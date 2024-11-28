# Modern FiveM Dealership UI V2

An enhanced version of the dealership system with improved features, better integration with ox_core, and advanced vehicle management capabilities.

## Features

- 🚗 Support for multiple vehicle types (New Cars, Used Cars, VIP Vehicles)
- 🎨 Modern, responsive design with smooth animations
- 🔍 Advanced filtering and sorting options
- 📦 Stock management system with real-time updates
- 🎮 Full ox_core integration
- 🕹️ Test drive functionality with timer
- 📊 Admin panel for inventory and sales management
- 🏷️ Dynamic pricing and stock control
- 🔐 VIP vehicle access control
- 📝 Detailed sales history tracking
- 🚙 Used vehicle system with mileage tracking
- ⚙️ Highly configurable through config files

## Installation

1. Copy this resource to your FiveM server's resources directory
2. Import `dealership.sql` to your database
3. Add the following to your server.cfg:
   ```cfg
   ensure ox_lib
   ensure ox_core
   ensure CarDealershipV2
   ```
4. Configure the dealership by editing `config.js`
5. Add your vehicle images to the `images/cars` directory

## Database Structure

The dealership uses a dedicated database structure for inventory and sales:

1. `dealership_inventory`: Stores all available vehicles and their details
   - Stock management
   - Vehicle specifications
   - Used vehicle tracking
   - Price management

2. `dealership_sales`: Tracks all vehicle sales
   - Purchase history
   - Buyer information
   - Price paid
   - Vehicle identifiers (plate/VIN)

See `dealership.sql` for the complete database schema.

## Configuration

The `config.js` file allows you to customize various aspects of the dealership:

```javascript
{
    dealershipName: 'Your Dealership Name',
    defaultType: 'Cars',
    currency: '$',
    
    ui: {
        showStock: true,
        showDescription: true,
        enablePriceFilter: true,  // Price filter with exact value input (auto-rounds to whole numbers)
        enableStockFilter: true,
        enableSearch: true,
        cardsPerRow: {
            sm: 1,  // mobile
            md: 2,  // tablet
            lg: 3,  // desktop
            xl: 4   // large desktop
        }
    },
    
    vehicleTypes: [
        {
            id: 'Cars',
            label: 'Cars',
            icon: 'fa-car'
        },
        {
            id: 'Used',
            label: 'Used Vehicles',
            icon: 'fa-car-side'
        },
        // Add more vehicle types
    ],
    
    vehicles: [
        {
            id: 'unique_id',
            type: 'Cars',
            name: 'Vehicle Name',
            price: 50000,
            stock: 5,
            image: './images/vehicle.jpg',
            description: 'Vehicle description',
            spawnCode: 'vehicle_spawn_code',
            specs: {
                topSpeed: '155 mph',
                acceleration: '5.1s 0-60',
                handling: '8.5/10',
                mileage: '0 km' // Only shown for used vehicles
            }
        },
        // Add more vehicles
    ]
}
```

## FiveM Integration

### Client-side Events

```lua
-- Open the dealership UI
TriggerEvent('dealership:open', 'Cars') -- Optional type parameter

-- Close the dealership UI
TriggerEvent('dealership:close')

-- Update vehicle stock
TriggerEvent('dealership:updateStock', {
    vehicleId = 'car_1',
    amount = -1 -- Negative to decrease, positive to increase
})

-- Update vehicle price
TriggerEvent('dealership:updatePrice', {
    vehicleId = 'car_1',
    price = 75000
})
```

### NUI Callbacks

The UI sends the following events that you need to handle in your client script:

```lua
RegisterNUICallback('purchaseVehicle', function(data, cb)
    -- data contains:
    -- vehicleId: string
    -- spawnCode: string
    -- price: number
    
    -- Your purchase logic here
    
    cb({})
end)

RegisterNUICallback('close', function(data, cb)
    -- Hide the NUI cursor and UI
    SetNuiFocus(false, false)
    cb({})
end)

RegisterNUICallback('typeChanged', function(data, cb)
    -- data contains:
    -- type: string (Cars, Boats, etc.)
    
    -- Your type change logic here
    
    cb({})
end)
```

## Debug Commands

When testing in a browser, the following debug commands are available in the console:

```javascript
// Add or remove stock
dealership.addStock('car_1', 1)  // Add 1 stock
dealership.addStock('car_1', -1) // Remove 1 stock

// Update vehicle price
dealership.setPrice('car_1', 75000)

// Add a new vehicle
dealership.addVehicle(
    'Cars',                  // type
    'New Vehicle',           // name
    50000,                  // price
    5,                      // stock
    './images/vehicle.jpg', // image
    'Description',          // description
    'spawn_code',           // FiveM spawn code
    {                       // specs (optional)
        topSpeed: '155 mph',
        acceleration: '5.1s 0-60',
        handling: '8.5/10'
    }
)

// Open specific vehicle type
dealership.openType('Cars')
```

## Customization

### Adding New Vehicle Types

1. Add the new type to `vehicleTypes` in `config.js`
2. Add vehicles of that type to the `vehicles` array
3. Use Font Awesome icons for the type by setting the `icon` property

### Styling

The UI uses Tailwind CSS for styling. You can customize the appearance by:

1. Modifying classes in `index.html`
2. Adding custom styles to `styles.css`
3. Updating the color scheme in the Tailwind classes

## License

This resource is licensed under the MIT License. Feel free to modify and share it!