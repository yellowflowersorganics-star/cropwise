# 🏢 Unit Management API Documentation

## Overview

Units represent physical locations/buildings within an organization. Each unit has:
- One Raspberry Pi Gateway
- Multiple Zones (rooms)
- Multiple ESP32 controllers (one per zone)

---

## Endpoints

### 1. Get All Units

**GET** `/api/units`

Returns all units for the authenticated organization.

**Headers**:
```
Authorization: Bearer <jwt_token>
```

**Response**:
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "organizationId": "uuid",
      "name": "Building A",
      "unitCode": "BLD-A",
      "unitType": "building",
      "gatewayId": "rpi_b827eb123456",
      "gatewayStatus": "online",
      "gatewayLastSeen": "2024-01-15T10:30:00Z",
      "totalZones": 5,
      "activeZones": 4,
      "status": "active",
      "zones": [
        {
          "id": "uuid",
          "name": "Room 1",
          "status": "running"
        }
      ]
    }
  ],
  "count": 1
}
```

---

### 2. Get Single Unit

**GET** `/api/units/:id`

Get detailed information about a specific unit.

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Building A",
    "unitCode": "BLD-A",
    "location": {
      "address": "123 Farm Road",
      "city": "Pune",
      "coordinates": { "lat": 18.5204, "lng": 73.8567 }
    },
    "unitType": "building",
    "totalArea": 500,
    "gatewayId": "rpi_b827eb123456",
    "gatewayStatus": "online",
    "gatewayLastSeen": "2024-01-15T10:30:00Z",
    "gatewayIpAddress": "192.168.1.100",
    "gatewayVersion": "1.2.3",
    "localMqttBroker": "192.168.1.100",
    "localMqttPort": 1883,
    "networkConfig": {
      "ssid": "Unit_Network",
      "dhcp": true
    },
    "contactPerson": "John Doe",
    "contactPhone": "+91-9876543210",
    "zones": [
      {
        "id": "uuid",
        "name": "Room 1",
        "zoneNumber": "A1",
        "status": "running",
        "devices": [
          {
            "id": "uuid",
            "deviceId": "esp32_aabbccddeeff",
            "deviceType": "esp32_controller",
            "status": "online"
          }
        ]
      }
    ],
    "devices": [
      {
        "id": "uuid",
        "deviceId": "rpi_b827eb123456",
        "deviceType": "raspberry_pi_gateway",
        "status": "online"
      }
    ]
  }
}
```

---

### 3. Create Unit

**POST** `/api/units`

Create a new unit.

**Request Body**:
```json
{
  "name": "Building B",
  "unitCode": "BLD-B",
  "location": {
    "address": "456 Farm Road",
    "city": "Mumbai",
    "coordinates": { "lat": 19.0760, "lng": 72.8777 }
  },
  "unitType": "building",
  "totalArea": 750,
  "gatewayId": "rpi_b827eb789012",
  "networkConfig": {
    "ssid": "Building_B_Network",
    "dhcp": true
  },
  "contactPerson": "Jane Smith",
  "contactPhone": "+91-9876543211"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "organizationId": "uuid",
    "name": "Building B",
    "unitCode": "BLD-B",
    "gatewayId": "rpi_b827eb789012",
    "gatewayStatus": "offline",
    "status": "active",
    "createdAt": "2024-01-15T10:35:00Z"
  },
  "message": "Unit created successfully"
}
```

---

### 4. Update Unit

**PUT** `/api/units/:id`

Update unit details.

**Request Body**:
```json
{
  "name": "Building B - Updated",
  "contactPerson": "Jane Doe",
  "status": "maintenance"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Building B - Updated",
    "contactPerson": "Jane Doe",
    "status": "maintenance"
  },
  "message": "Unit updated successfully"
}
```

---

### 5. Delete Unit (Decommission)

**DELETE** `/api/units/:id`

Soft delete a unit by setting status to 'decommissioned'.

**Response**:
```json
{
  "success": true,
  "message": "Unit decommissioned successfully"
}
```

**Error (if active zones exist)**:
```json
{
  "success": false,
  "error": "Cannot delete unit with 3 active zones. Stop all zones first."
}
```

---

### 6. Gateway Heartbeat

**POST** `/api/units/:id/gateway-heartbeat`

Called by Raspberry Pi gateway to report status.

**Headers**:
```
Authorization: Bearer <gateway_jwt_token>
```

**Request Body**:
```json
{
  "gatewayId": "rpi_b827eb123456",
  "ipAddress": "192.168.1.100",
  "version": "1.2.3",
  "zoneStatuses": [
    {
      "esp32Id": "esp32_aabbccddeeff",
      "status": "online"
    },
    {
      "esp32Id": "esp32_112233445566",
      "status": "online"
    }
  ],
  "systemInfo": {
    "cpuTemp": 45.2,
    "memoryUsage": 62.5,
    "diskUsage": 35.8,
    "uptime": 86400
  }
}
```

**Response**:
```json
{
  "success": true,
  "message": "Heartbeat received",
  "timestamp": "2024-01-15T10:40:00Z"
}
```

---

### 7. Unit Statistics

**GET** `/api/units/:id/statistics`

Get comprehensive statistics for a unit.

**Response**:
```json
{
  "success": true,
  "data": {
    "unit": {
      "id": "uuid",
      "name": "Building A",
      "status": "active",
      "totalZones": 5,
      "activeZones": 4
    },
    "gateway": {
      "status": "online",
      "online": true,
      "lastSeen": "2024-01-15T10:40:00Z",
      "version": "1.2.3"
    },
    "zones": [
      {
        "status": "running",
        "count": 4
      },
      {
        "status": "stopped",
        "count": 1
      }
    ],
    "devices": [
      {
        "status": "online",
        "deviceType": "esp32_controller",
        "count": 5
      },
      {
        "status": "online",
        "deviceType": "raspberry_pi_gateway",
        "count": 1
      }
    ]
  }
}
```

---

## Unit Status Values

| Status | Description |
|:-------|:------------|
| `active` | Unit is operational |
| `inactive` | Unit is temporarily disabled |
| `maintenance` | Unit is under maintenance |
| `decommissioned` | Unit is permanently removed |

## Gateway Status Values

| Status | Description |
|:-------|:------------|
| `online` | Gateway connected (heartbeat < 5 min) |
| `offline` | Gateway disconnected |
| `maintenance` | Gateway being serviced |
| `error` | Gateway reporting errors |

## Unit Types

| Type | Description |
|:-----|:------------|
| `building` | General building |
| `farm` | Agricultural facility |
| `warehouse` | Storage facility |
| `facility` | Processing facility |
| `greenhouse` | Greenhouse structure |

---

## Usage Examples

### Setup New Location

```javascript
// 1. Create unit
const unit = await fetch('/api/units', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer ' + token,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'Main Farm',
    unitType: 'farm',
    gatewayId: 'rpi_mac_address'
  })
});

// 2. Add zones
const zone1 = await fetch('/api/zones', {
  method: 'POST',
  body: JSON.stringify({
    unitId: unit.id,
    name: 'Incubation Room 1',
    zoneNumber: 'A1'
  })
});

// 3. Register ESP32
const device = await fetch('/api/devices', {
  method: 'POST',
  body: JSON.stringify({
    unitId: unit.id,
    zoneId: zone1.id,
    deviceId: 'esp32_mac_address',
    deviceType: 'esp32_controller'
  })
});
```

### Monitor Gateway Health

```javascript
// Raspberry Pi sends heartbeat every minute
setInterval(async () => {
  await fetch(`/api/units/${unitId}/gateway-heartbeat`, {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer ' + gatewayToken,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      gatewayId: 'rpi_b827eb123456',
      ipAddress: getLocalIP(),
      version: '1.2.3',
      zoneStatuses: await getZoneStatuses(),
      systemInfo: await getSystemInfo()
    })
  });
}, 60000); // Every 60 seconds
```

---

## Error Codes

| Code | Message | Description |
|:-----|:--------|:------------|
| 400 | Unit name is required | Missing name field |
| 400 | Gateway ID already registered | Gateway belongs to another unit |
| 400 | Cannot delete unit with active zones | Zones must be stopped first |
| 404 | Unit not found | Invalid unit ID or not in org |
| 500 | Failed to create unit | Database error |

---

## Notes

1. **Gateway Registration**: When you provide `gatewayId` during unit creation, a Device record is automatically created for the Raspberry Pi.

2. **Multi-Tenancy**: All queries are scoped to the authenticated organization.

3. **Gateway Heartbeat**: Gateways should send heartbeat every 60 seconds. If no heartbeat for 5 minutes, status changes to 'offline'.

4. **Zone Counts**: `totalZones` and `activeZones` are automatically calculated and updated.

5. **Soft Delete**: Units are never hard-deleted. They're marked as 'decommissioned' for audit trail.

---

**Next**: [Zone Management API](./API_ZONES.md)

