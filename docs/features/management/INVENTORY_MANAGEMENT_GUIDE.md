# Inventory Management System

## 📦 Overview

The Inventory Management system provides comprehensive tracking of all growing supplies, including substrate, spawn, consumables, packaging, chemicals, and equipment.

## ✨ Features

### 1. **Inventory Item Management**
- Add, edit, and delete inventory items
- Categorize items (substrate, spawn, consumables, packaging, chemicals, equipment, other)
- Track current stock levels
- Set minimum and maximum stock thresholds
- Cost tracking per unit and total value calculation
- Supplier information and contact details
- Storage location tracking
- Expiry date monitoring

### 2. **Stock Management**
- Real-time stock level tracking
- Multiple adjustment types:
  - **Purchase**: Add stock from suppliers
  - **Usage**: Remove stock for production
  - **Waste**: Track damaged or lost items
  - **Manual Adjustments**: Add or remove for corrections
  - **Transfer**: Move stock between locations
  - **Return**: Return unused stock from batches

### 3. **Transaction History**
- Complete audit trail of all stock movements
- Track who made changes and when
- Link transactions to batches and zones
- Filter by date range, item, or transaction type

### 4. **Low Stock Alerts**
- Automatic detection of items below minimum threshold
- Dashboard alert banner showing low stock items
- Visual indicators on inventory cards
- Filter view for low stock items only

### 5. **Search and Filtering**
- Search by item name, SKU, or supplier
- Filter by category
- Filter by active status
- Low stock quick filter

### 6. **Analytics**
- Total inventory count
- Total inventory value
- Low stock item count
- Category-wise breakdown

## 🏗️ Database Schema

### InventoryItem Model
```javascript
{
  id: UUID,
  ownerId: UUID,
  organizationId: UUID (optional),
  name: String,
  category: ENUM [substrate, spawn, consumables, packaging, chemicals, equipment, other],
  sku: String,
  description: Text,
  unit: String (kg, lb, bag, bottle, piece, etc.),
  currentStock: Float,
  minStockLevel: Float,
  maxStockLevel: Float,
  unitCost: Decimal,
  totalValue: Decimal (auto-calculated),
  supplier: String,
  supplierContact: String,
  location: String,
  expiryDate: Date,
  lastRestocked: Date,
  isActive: Boolean,
  notes: Text,
  metadata: JSON
}
```

### InventoryTransaction Model
```javascript
{
  id: UUID,
  itemId: UUID (FK to inventory_items),
  ownerId: UUID,
  organizationId: UUID (optional),
  type: ENUM [purchase, usage, waste, adjustment_add, adjustment_remove, transfer, return],
  quantity: Float (positive for add, negative for remove),
  previousStock: Float,
  newStock: Float,
  unitCost: Decimal,
  totalCost: Decimal,
  batchId: String (FK to batches),
  zoneId: UUID (FK to zones),
  notes: Text,
  metadata: JSON,
  createdAt: Date
}
```

## 🔌 API Endpoints

### Inventory Items
```http
GET    /api/inventory/items              # Get all items (with filtering)
GET    /api/inventory/items/:id          # Get single item
POST   /api/inventory/items              # Create item
PUT    /api/inventory/items/:id          # Update item
DELETE /api/inventory/items/:id          # Delete (soft delete) item
GET    /api/inventory/items/low-stock    # Get low stock items
```

### Stock Adjustments
```http
POST   /api/inventory/items/:id/adjust   # Adjust stock level
```

### Transactions
```http
GET    /api/inventory/transactions       # Get transaction history
```

### Usage Tracking
```http
POST   /api/inventory/usage              # Record batch usage
```

### Statistics
```http
GET    /api/inventory/stats              # Get inventory stats
```

## 📝 Usage Examples

### Creating an Inventory Item

```javascript
const newItem = {
  name: "Oyster Mushroom Spawn",
  category: "spawn",
  sku: "OMS-001",
  unit: "kg",
  currentStock: 50,
  minStockLevel: 10,
  maxStockLevel: 100,
  unitCost: 500,
  supplier: "Mushroom Spawn Suppliers Pvt Ltd",
  supplierContact: "+91-9876543210",
  location: "Cold Room 1",
  notes: "Store at 4°C"
};

await inventoryService.create(newItem);
```

### Adjusting Stock (Purchase)

```javascript
await inventoryService.adjustStock(itemId, {
  quantity: 25,  // Positive for adding
  type: 'purchase',
  notes: 'Restocked from supplier - Invoice #12345'
});
```

### Adjusting Stock (Usage)

```javascript
await inventoryService.adjustStock(itemId, {
  quantity: -10,  // Negative for removing
  type: 'usage',
  batchId: 'ZONE-A1-2024-001',
  notes: 'Used in batch production'
});
```

### Recording Batch Usage

```javascript
await inventoryService.recordUsage({
  batchId: 'ZONE-A1-2024-001',
  zoneId: 'abc-123-def-456',
  items: [
    { itemId: 'spawn-id', quantity: 5, notes: 'Oyster spawn' },
    { itemId: 'substrate-id', quantity: 50, notes: 'Straw substrate' },
    { itemId: 'bag-id', quantity: 100, notes: 'Growing bags' }
  ],
  notes: 'Initial batch setup'
});
```

## 🎨 UI Components

### Inventory List Page (`/inventory`)
- Stats cards (total items, total value, low stock count, categories)
- Search bar
- Category filter
- Low stock filter toggle
- Item cards with:
  - Category icon
  - Stock level
  - Progress bar
  - Total value
  - Supplier info
  - Action buttons (Adjust Stock, Edit, Delete)

### Add/Edit Item Modal
- Full form for item creation/editing
- Category selection
- Stock level inputs (with min/max thresholds)
- Cost tracking
- Supplier information
- Location and notes

### Stock Adjustment Modal
- Transaction type selector
- Quantity input (with +/- indicator)
- Notes field
- Preview of new stock level
- Validation for insufficient stock

### Low Stock Alert (Dashboard)
- Alert banner showing count of low stock items
- List of up to 3 items with current and minimum levels
- Link to filtered inventory view
- Dismissible

## 🔗 Integration with Batch Management

### Automatic Stock Tracking
When starting a batch, you can optionally record inventory usage:

1. User starts a batch
2. System prompts to record inventory usage (optional)
3. User selects items and quantities
4. System automatically:
   - Reduces stock levels
   - Creates usage transactions
   - Links transactions to batch
   - Updates last usage date

### Transaction Linking
- All inventory transactions can be linked to specific batches
- View inventory usage per batch in batch history
- Track cost of goods per batch for profitability analysis

## 📊 Sample Data Categories

### Substrate Types
- Straw (bales, kg)
- Sawdust (bags, kg)
- Coco Coir (blocks, kg)
- Manure (bags, kg)
- Supplemented Mix (bags)

### Spawn/Culture
- Oyster Mushroom Spawn (kg, bags)
- Button Mushroom Spawn (kg, bags)
- Shiitake Spawn (kg, bags)
- Liquid Culture (bottles, ml)

### Consumables
- Growing Bags (pieces)
- Filter Patches (pieces)
- Gloves (boxes)
- Rubber Bands (kg)
- Labels (rolls)
- Tape (rolls)

### Packaging
- Harvest Containers (boxes)
- Retail Boxes (pieces)
- Vacuum Bags (rolls)
- Labels (sheets)

### Chemicals
- pH Down (bottles, liters)
- pH Up (bottles, liters)
- Disinfectant (bottles, liters)
- Sterilizer (kg, liters)

### Equipment
- Thermometers (pieces)
- Hygrometers (pieces)
- Knives (pieces)
- Scissors (pieces)

## 🔒 Access Control

### Single-Tenant Mode
- Items are filtered by `ownerId`
- Only the creating user can view/edit their items
- Transactions tracked by user

### Multi-Tenant Mode
- Items are filtered by `organizationId`
- All users in organization can view/edit
- Role-based permissions (future enhancement)

## 📈 Future Enhancements

### Phase 2
- [ ] Barcode/QR code generation and scanning
- [ ] Batch import/export (CSV/Excel)
- [ ] Supplier management portal
- [ ] Purchase order system
- [ ] Automatic reorder points
- [ ] Email notifications for low stock

### Phase 3
- [ ] Mobile app for stock counting
- [ ] Integration with accounting systems
- [ ] FIFO/LIFO cost calculation
- [ ] Expiry date tracking and alerts
- [ ] Warehouse location mapping
- [ ] Multi-currency support

### Phase 4
- [ ] AI-powered demand forecasting
- [ ] Optimal reorder quantity suggestions
- [ ] Price trend analysis
- [ ] Supplier performance tracking
- [ ] Integration with e-commerce platforms

## 🐛 Troubleshooting

### Low Stock Not Showing
- Check if `minStockLevel` is set for items
- Verify items are marked as `isActive`
- Refresh dashboard to reload alerts

### Stock Adjustment Failed
- Check for sufficient stock when removing
- Verify user has permission to modify item
- Check network connection

### Total Value Not Calculating
- Ensure both `currentStock` and `unitCost` are set
- Value auto-calculates on save
- Refresh item to see updated value

## 📞 Support

For issues or feature requests related to Inventory Management:
- Check this documentation
- Review API endpoint responses for error details
- Check backend logs for transaction failures

---

**Version:** 1.0.0  
**Last Updated:** November 2024  
**Module:** Inventory Management

