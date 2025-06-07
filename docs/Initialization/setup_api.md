# To initialize a vehicle

## Response status

| Status | Description |
|--------|------------------|
| 200 | Successful |
| 400 | Bad request |
| 409 | Conflict |

## To prepare the vehicle

1. Power on the vehicle and your computer.

2. Make sure that the vehicle and your computer are connected to the same network.

3. Do a check of all critical systems:

    ```bash title="Request"
    curl -X GET "http://api.muber.fleet:1234/v2/vehicles/veh_123/full-safety-check" \
         -H "Authorization: Bearer YOUR_API_KEY" \
         -H "Accept: application/json"
    ```

    ```json title="Response (status 200)"
    {
      "status": "success",
      "vehicle_id": "veh_123",
      "all_systems_go": true,
      "systems": {
        "brakes": {
          "status": "ok",
          "pressure_psi": 1200,
          "pad_wear": "15%"
        },
        "cameras_sensors": {
          "status": "ok",
          "faulty_sensors": [],
          "camera_obstructions": 0
        }
      },
      "timestamp": "2025-06-10T14:30:00Z"
    }
    ```

    ```json title="Response (status 400)"
    {
      "status": "error",
      "code": "INVALID_REQUEST",
      "message": "Missing required parameters"
    }
    ```

    !!! warning
        Do not register the vehicle if one of the systems is not working properly.

## To assign a **Fleet ID** to a vehicle

1. Do a check to see if the vehicle is already registered.

    ```bash title="Request"
    curl -X GET "http://api.muber.fleet:1234/v2/monitor" \
         -H "Authorization: Bearer YOUR_API_KEY"
    ```

    ```json title="Response (status 200)"
    {
      "status": "success",
      "data": [
        {
          "vehicle_id": "veh_001",
          "status": "active",
          "location": {"lat": 34.0522, "lng": -118.2437},
          "last_updated": "2025-06-07T12:30:00Z"
        }
      ]
    }
    ```

    ```json title="Response (status 400)"
    {
    "status": "error",
    "code": "MISSING_IDENTIFIERS",
    "message": "At least one identifier (vin, mac_address) must be provided"
    }
    ```

2. If the vehicle is not registered, send the request that follows to register it.

    ``` bash title="Request"
    curl -X POST "http://api.muber.fleet:1234/v2/vehicles" \
         -H "Authorization: Bearer YOUR_API_KEY" \
         -H "Content-Type: application/json" \
         -d '{
            "vehicle_id": "veh_002",
            "model": "Ford Transit",
            "year": 2024,
            "initial_status": "inactive"
            }'
    ```

    ```json title="Request (status 200)"
    {
      "status": "success",
      "vehicle_id": "veh_002",
      "message": "Vehicle registered successfully."
    }
    ```

4. Make sure that over-the-air software updates are encrypted and authenticated.

## To select the operating mode

Only available when you use the [GUI](../Initialization/setup.md).

## To calibrate the vehicle according to terrain type

1. Make sure that the vehicle is powered and stopped.

    ```bash title="Request"
    curl -X GET "http://api.muber.fleet:1234/v2/vehicles/veh_123/safety-check" \
         -H "Authorization: Bearer YOUR_API_KEY" \
         -H "Accept: application/json"
    ```
    
    ```json title="Response (status 200)"
    {
      "status": "success",
      "vehicle_id": "veh_123",
      "is_safe": true,
      "details": {
        "speed_kmh": 0,
        "ignition_status": "off",
        "gear_position": "park",
        "last_updated": "2025-06-10T14:30:00Z"
      }
    }
    ```

    ```json title="Response (status 409)"
    {
      "status": "error",
      "vehicle_id": "veh_123",
      "is_safe": false,
      "errors": [
        {
          "code": "VEHICLE_MOVING",
          "message": "Vehicle speed is 12 km/h (must be 0)."
        },
        {
          "code": "IGNITION_ON",
          "message": "Ignition is still active."
        }
      ]
    }
    ```

2. Select the terrain type.

    ```bash title="Request"
    curl -X POST "http://api.muber.fleet:1234/v2/vehicles/veh_123/terrain" \
          -H "Authorization: Bearer YOUR_API_KEY" \
          -H "Content-Type: application/json" \
          -d '{"terrain_type": "off_road"}'
    ```

    ```json title="Response (status 200)"
    {
    "status": "success",
    "vehicle_id": "veh_123",
    "new_terrain": "off_road",
    "system_adjustments": {
      "suspension": "high_clearance",
      "traction_control": "aggressive",
      "camera_processing": "obstacle_priority"
    },
    "confirmation_required": false
    }
    ```

    ```json title="Response (status 400)"
    {
      "status": "error",
      "error_code": "INVALID_TERRAIN",
      "valid_options": ["urban", "suburban", "off_road"],
      "received": "desert" 
    }
    ```

    !!! note
        The vehicle can take up to 15 seconds to complete the calibration.

    !!! note

        For example, an SUV configured for **off-road** might have higher suspension and
        different route optimization.

3. Calibrate the vehicle sensors. See the [maintenance](../Maintenance/tips.md) section. 