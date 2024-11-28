# Modern FiveM Dealership UI V2 - Developer Documentation

This document provides in-depth technical documentation for developers working with or modifying the Modern FiveM Dealership UI V2 resource.

## Architecture Overview

The dealership UI V2 is built using a modern web stack that integrates with FiveM's NUI system and ox_core. Here's a breakdown of the key components:

### Frontend Stack
- **Framework**: Vanilla JavaScript for lightweight performance
- **Styling**: Tailwind CSS for utility-first styling
- **Build System**: Webpack for asset bundling and optimization
- **State Management**: Custom event-driven state management system
- **Image Loading**: Progressive image loading for vehicle previews

### Backend Integration
- **FiveM NUI**: Browser-based UI system
- **ox_core**: Core game functionality and player data
- **ox_lib**: UI components and callbacks
- **Event System**: Bidirectional communication between UI and game
- **Database**: MySQL for inventory and sales tracking

## Component Architecture

### Core Components

#### 1. DealershipManager
The central controller that manages the dealership state and coordinates between components.

```javascript
class DealershipManager {
    constructor(config) {
        this.vehicles = new Map();
        this.currentType = config.defaultType;
        this.filters = new FilterManager();
        this.eventBus = new EventBus();
        this.init();
    }
    
    init() {
        this.loadVehicles();
        this.setupEventListeners();
    }
    
    // ... additional methods
}
```

#### 2. FilterManager
Handles all filtering and sorting logic for vehicles.

```javascript
class FilterManager {
    constructor() {
        this.activeFilters = new Set();
        this.sortOrder = 'asc';
        this.sortField = 'price';
    }
    
    applyFilters(vehicles) {
        return vehicles.filter(vehicle => 
            this.activeFilters.size === 0 || 
            this.matchesFilters(vehicle)
        );
    }
    
    validatePriceRange(type, value) {
        // Ensure values are numbers and round appropriately
        // Min price rounds down, max price rounds up for inclusive range
        if (type === 'min') {
            return Math.floor(Number(value) || 0);
        } else {
            return Math.ceil(Number(value) || this.maxPrice);
        }
    }
    
    // ... additional methods
}
```

#### 3. EventBus
Custom event system for component communication.

```javascript
class EventBus {
    constructor() {
        this.listeners = new Map();
    }
    
    on(event, callback) {
        if (!this.listeners.has(event)) {
            this.listeners.set(event, new Set());
        }
        this.listeners.get(event).add(callback);
    }
    
    emit(event, data) {
        if (this.listeners.has(event)) {
            this.listeners.get(event).forEach(callback => callback(data));
        }
    }
}
```

## State Management

### Debug Mode
By default, all vehicle types are available when the index starts for debugging purposes. This can be controlled through the `DEBUG` constant:

```javascript
const DEBUG = {
    SHOW_ALL_TYPES: true,  // Show all vehicle types on startup
    LOG_EVENTS: true,      // Log all events to console
    MOCK_NETWORK: false    // Use mock data instead of real network calls
};
```

### Vehicle Types
The system supports both new and used vehicles. Used vehicles have additional properties like mileage that are displayed in the UI:

```javascript
const VehicleTypes = {
    NEW: 'Cars',
    USED: 'Used',
    BOATS: 'Boats',
    PLANES: 'Planes',
    MOTORCYCLES: 'Motorcycles'
};
```

### Used Vehicle Management
Used vehicles have additional functionality for mileage tracking:

```javascript
class UsedVehicleManager {
    constructor() {
        this.mileageUpdateInterval = 1000; // 1 second
        this.activeVehicles = new Map();
    }
    
    updateMileage(vehicleId, newMileage) {
        const vehicle = this.activeVehicles.get(vehicleId);
        if (vehicle) {
            vehicle.specs.mileage = `${newMileage} km`;
            this.emit('mileageUpdated', { vehicleId, mileage: newMileage });
        }
    }
    
    startMileageTracking(vehicleId) {
        // Called when a used vehicle is spawned
        const vehicle = this.activeVehicles.get(vehicleId);
        if (vehicle) {
            this.trackingIntervals.set(vehicleId, setInterval(() => {
                // Update mileage based on vehicle usage
                const currentMileage = parseInt(vehicle.specs.mileage);
                this.updateMileage(vehicleId, currentMileage + 1);
            }, this.mileageUpdateInterval));
        }
    }
    
    stopMileageTracking(vehicleId) {
        const interval = this.trackingIntervals.get(vehicleId);
        if (interval) {
            clearInterval(interval);
            this.trackingIntervals.delete(vehicleId);
        }
    }
}
```

### Vehicle State
Vehicles are managed using a Map for O(1) lookups and efficient updates:

```javascript
{
    vehicles: Map<string, {
        id: string,
        type: string,
        name: string,
        price: number,
        stock: number,
        image: string,
        description: string,
        spawnCode: string,
        specs: {
            topSpeed: string,
            acceleration: string,
            handling: string
        }
    }>
}
```

### UI State
The UI state is managed through a reactive system that updates components when data changes:

```javascript
class UIState {
    constructor() {
        this.observers = new Set();
        this.state = {
            selectedType: null,
            filters: [],
            searchQuery: '',
            sortOrder: 'asc',
            sortField: 'price'
        };
    }
    
    setState(newState) {
        this.state = { ...this.state, ...newState };
        this.notifyObservers();
    }
    
    subscribe(observer) {
        this.observers.add(observer);
    }
    
    // ... additional methods
}
```

## FiveM Integration Details

### NUI Message Protocol
Messages between the game and UI follow this structure:

```typescript
interface NUIMessage {
    type: string;
    payload: {
        action: string;
        data: any;
    };
}
```

### Event Flow
1. **Game to UI**:
```lua
-- Client script
SendNUIMessage({
    type = "update",
    payload = {
        action = "updateStock",
        data = {
            vehicleId = "car_1",
            newStock = 5
        }
    }
})
```

2. **UI to Game**:
```javascript
// JavaScript
fetch(`https://${GetParentResourceName()}/purchaseVehicle`, {
    method: 'POST',
    body: JSON.stringify({
        vehicleId: 'car_1',
        spawnCode: 'adder',
        price: 50000
    })
});
```

## Performance Optimizations

### 1. Virtual List Rendering
For large vehicle lists, we implement virtual scrolling:

```javascript
class VirtualList {
    constructor(container, itemHeight) {
        this.container = container;
        this.itemHeight = itemHeight;
        this.visibleItems = new Map();
        this.totalItems = 0;
        this.scrollTop = 0;
        this.viewportHeight = 0;
        this.setupScroll();
    }
    
    updateVisibleItems() {
        const startIndex = Math.floor(this.scrollTop / this.itemHeight);
        const endIndex = Math.min(
            startIndex + Math.ceil(this.viewportHeight / this.itemHeight) + 1,
            this.totalItems
        );
        
        // Update only necessary items
        for (let i = startIndex; i <= endIndex; i++) {
            if (!this.visibleItems.has(i)) {
                this.renderItem(i);
            }
        }
        
        // Remove items that are no longer visible
        for (const [index] of this.visibleItems) {
            if (index < startIndex || index > endIndex) {
                this.removeItem(index);
            }
        }
    }
}
```

### 2. Image Optimization
Images are lazy-loaded and cached:

```javascript
class ImageLoader {
    constructor() {
        this.cache = new Map();
        this.observer = new IntersectionObserver(this.onIntersection.bind(this));
    }
    
    preloadImage(src) {
        if (this.cache.has(src)) return this.cache.get(src);
        
        const promise = new Promise((resolve, reject) => {
            const img = new Image();
            img.onload = () => resolve(img);
            img.onerror = reject;
            img.src = src;
        });
        
        this.cache.set(src, promise);
        return promise;
    }
}
```

## Debug Tools

### Vehicle Type Display
In debug mode, all vehicle types are shown by default. This helps developers test different vehicle categories without needing to trigger specific conditions:

```javascript
class DealershipDebug {
    static initializeTypes(dealership) {
        if (DEBUG.SHOW_ALL_TYPES) {
            // Show all vehicle types immediately
            dealership.showAllTypes();
        } else {
            // Normal initialization - types shown based on conditions
            dealership.initializeDefaultTypes();
        }
    }
    
    static mockUsedVehicle(baseVehicle) {
        return {
            ...baseVehicle,
            type: 'Used',
            specs: {
                ...baseVehicle.specs,
                mileage: `${Math.floor(Math.random() * 100000)} km`
            },
            price: Math.floor(baseVehicle.price * 0.7) // 30% discount for used vehicles
        };
    }
}
```

### Console API
Extended debug commands for development:

```javascript
class DebugTools {
    static init(dealership) {
        window.dealership = {
            ...window.dealership,
            
            debug: {
                dumpState: () => console.log(dealership.getState()),
                simulateNetworkLatency: (ms) => {
                    dealership.setNetworkDelay(ms);
                },
                triggerError: (type) => {
                    dealership.simulateError(type);
                },
                reloadConfig: async () => {
                    await dealership.loadConfig();
                },
                createUsedVehicle: (baseVehicleId, mileage) => {
                    dealership.createUsedVariant(baseVehicleId, mileage);
                },
                updateMileage: (vehicleId, newMileage) => {
                    dealership.updateVehicleMileage(vehicleId, newMileage);
                }
            }
        };
    }
}
```

## Error Handling

### Error Types
```javascript
const ErrorTypes = {
    NETWORK: 'NETWORK_ERROR',
    VALIDATION: 'VALIDATION_ERROR',
    STOCK: 'STOCK_ERROR',
    PERMISSION: 'PERMISSION_ERROR'
};

class DealershipError extends Error {
    constructor(type, message, details = {}) {
        super(message);
        this.type = type;
        this.details = details;
        this.timestamp = new Date();
    }
}
```

### Error Handling Strategy
```javascript
class ErrorHandler {
    static handle(error) {
        console.error(`[Dealership Error] ${error.type}:`, error);
        
        switch (error.type) {
            case ErrorTypes.NETWORK:
                this.handleNetworkError(error);
                break;
            case ErrorTypes.STOCK:
                this.handleStockError(error);
                break;
            // ... handle other error types
        }
    }
    
    static handleNetworkError(error) {
        // Implement retry logic
        this.retryOperation(error.details.operation, 3);
    }
}
```

## Testing

### Unit Tests
Example test setup using Jest:

```javascript
describe('DealershipManager', () => {
    let manager;
    
    beforeEach(() => {
        manager = new DealershipManager(mockConfig);
    });
    
    test('should update vehicle stock correctly', () => {
        const vehicleId = 'car_1';
        const initialStock = manager.getVehicleStock(vehicleId);
        
        manager.updateStock(vehicleId, 1);
        expect(manager.getVehicleStock(vehicleId)).toBe(initialStock + 1);
    });
    
    // ... additional tests
});
```

## Contributing Guidelines

### Code Style
- Use ES6+ features appropriately
- Follow the existing event-driven architecture
- Maintain type safety with JSDoc comments
- Keep components focused and single-responsibility

### Pull Request Process
1. Branch naming: `feature/description` or `fix/description`
2. Include relevant unit tests
3. Update documentation for API changes
4. Ensure all CI checks pass

## Security Considerations

### Input Validation
All user inputs and NUI messages are validated:

```javascript
class Validator {
    static validateVehicleData(data) {
        const schema = {
            id: /^[a-zA-Z0-9_-]+$/,
            price: (value) => typeof value === 'number' && value >= 0,
            stock: (value) => Number.isInteger(value) && value >= 0
        };
        
        return this.validate(data, schema);
    }
    
    static validate(data, schema) {
        for (const [key, validator] of Object.entries(schema)) {
            if (!validator(data[key])) {
                throw new DealershipError(
                    ErrorTypes.VALIDATION,
                    `Invalid ${key}`,
                    { value: data[key] }
                );
            }
        }
    }
}
```

### XSS Prevention
All dynamic content is sanitized before rendering:

```javascript
function sanitizeHTML(str) {
    const div = document.createElement('div');
    div.textContent = str;
    return div.innerHTML;
}
```

## Optimization Tips

1. **Bundle Size**
   - Use code splitting for vehicle type modules
   - Implement tree-shaking for unused features
   - Compress images appropriately

2. **Runtime Performance**
   - Cache expensive computations
   - Use requestAnimationFrame for smooth animations
   - Implement debouncing for search/filter operations

3. **Memory Management**
   - Clean up event listeners
   - Implement proper garbage collection
   - Monitor memory usage in development

## License

This project is licensed under the MIT License. See the LICENSE file for details.