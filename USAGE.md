# CarDealershipV2 Usage Guide

This guide explains how to use the CarDealershipV2 system, both for players and administrators.

## For Players

### Accessing the Dealership
1. Visit the dealership location (marked on the map with a car icon)
2. Press `E` when near the dealership marker to open the UI
3. Browse available vehicles by category

### Vehicle Categories
- **New Cars**: Brand new vehicles at full price
- **Used Cars**: Pre-owned vehicles at discounted prices
- **VIP Vehicles**: Special vehicles for VIP players only

### Purchasing a Vehicle
1. Select a vehicle category
2. Browse available vehicles
3. Use filters to find specific vehicles:
   - Price range (enter exact values, automatically rounds to whole numbers)
   - Vehicle type
   - Stock availability
4. Click on a vehicle to view details:
   - Specifications
   - Price
   - Stock level
   - Description
5. Options available:
   - Purchase: Buy the vehicle immediately
   - Test Drive: Try the vehicle for 5 minutes
   - View 360°: Examine the vehicle from all angles

### Test Drive System
1. Click "Test Drive" on any vehicle
2. A timer will appear showing remaining test time
3. Vehicle will automatically despawn after 5 minutes
4. You can end the test drive early by returning to the dealership

### VIP Vehicles
1. Must have VIP status to purchase
2. Special vehicles with unique features
3. Exclusive customization options
4. Priority service

### Used Vehicles
1. Discounted prices
2. Mileage tracking
3. Previous owner history
4. Condition reports

## For Administrators

### Accessing Admin Panel
1. Use `/dealership admin` command
2. Must have admin permissions

### Inventory Management
1. Add new vehicles:
   ```lua
   exports.CarDealershipV2:AddVehicle({
       model = "adder",
       name = "Adder",
       price = 50000,
       category = "Cars",
       stock = 10,
       description = "Luxury sports car",
       specs = {
           topSpeed = "120 mph",
           acceleration = "3.2s 0-60",
           handling = "9/10"
       }
   })
   ```

2. Update stock levels:
   ```lua
   exports.CarDealershipV2:UpdateStock(vehicleId, amount)
   ```

3. Modify prices:
   ```lua
   exports.CarDealershipV2:UpdatePrice(vehicleId, newPrice)
   ```

### Sales Management
1. View sales history:
   ```lua
   exports.CarDealershipV2:GetSales({
       startDate = "2023-01-01",
       endDate = "2023-12-31",
       category = "Cars"
   })
   ```

2. Generate reports:
   - Daily sales
   - Popular vehicles
   - Revenue analysis
   - Stock levels

### Used Vehicle Management
1. Create used vehicles:
   ```lua
   exports.CarDealershipV2:CreateUsedVehicle({
       baseModel = "adder",
       mileage = 50000,
       priceMultiplier = 0.7
   })
   ```

2. Update mileage:
   ```lua
   exports.CarDealershipV2:UpdateMileage(vehicleId, newMileage)
   ```

### VIP Management
1. Set VIP vehicle status:
   ```lua
   exports.CarDealershipV2:SetVIPStatus(vehicleId, true)
   ```

2. Manage VIP access:
   ```lua
   exports.CarDealershipV2:UpdateVIPAccess(playerId, true)
   ```

## Common Commands

### Player Commands
- `/dealership` - Open dealership UI
- `/testdrive` - End current test drive
- `/vipstatus` - Check VIP status

### Admin Commands
- `/dealership admin` - Open admin panel
- `/dealership addstock <id> <amount>` - Add stock
- `/dealership setprice <id> <price>` - Set price
- `/dealership addvip <id>` - Add VIP vehicle
- `/dealership sales` - View sales history
- `/dealership inventory` - View inventory

## Troubleshooting

### Common Issues

1. **Can't Purchase Vehicle**
   - Check money balance
   - Verify stock availability
   - Confirm VIP status if needed
   - Check database connection

2. **Test Drive Issues**
   - Ensure no active test drive
   - Check spawn location clearance
   - Verify vehicle model exists
   - Check for script errors

3. **Admin Panel Access**
   - Verify admin permissions
   - Check group assignment
   - Confirm resource is running

4. **Stock Issues**
   - Check database connection
   - Verify transaction logs
   - Confirm stock numbers
   - Check for race conditions

### Error Messages

1. "Not enough money"
   - Player lacks sufficient funds
   - Check price and balance

2. "Vehicle not in stock"
   - Stock level is 0
   - Check inventory management

3. "VIP access required"
   - Player lacks VIP status
   - Check group assignments

4. "Transaction failed"
   - Database error
   - Check server logs
   - Verify MySQL connection

## Best Practices

1. **For Players**
   - Always test drive before purchase
   - Check vehicle specifications
   - Compare prices across categories
   - Read vehicle descriptions

2. **For Admins**
   - Regularly check stock levels
   - Monitor sales patterns
   - Keep prices updated
   - Maintain VIP vehicle exclusivity
   - Back up database regularly

3. **Resource Management**
   - Monitor server performance
   - Check error logs
   - Update vehicle images
   - Maintain database indexes

## Support

For technical support:
1. Check server logs
2. Review error messages
3. Contact resource maintainer
4. Check GitHub issues
5. Review documentation

## License

This resource is licensed under the MIT License. See LICENSE file for details.