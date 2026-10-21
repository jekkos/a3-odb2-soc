# A3 PHEV HV SOC - Home Assistant Add-on

Reads HV SOC and available energy from Audi A3 PHEV via BLE ELM327 and exposes it to Home Assistant via MQTT.

## Features

- Reads HV battery SOC from Audi A3 PHEV (8Y) via BLE ELM327 adapter
- Publishes SOC percentage to `car/a3/hv_soc`
- Publishes estimated available energy (kWh) to `car/a3/hv_kwh`
- Automatic MQTT broker discovery (built-in or external)
- Self-healing: restarts on failure with clear error messages

## Requirements

- Home Assistant with Add-on Store (HAOS, Supervised, or Docker)
- MQTT broker (built-in or external)
- BLE ELM327 adapter (tested with "S1 Max" BLE)
- Audi A3 PHEV (8Y) with OBD port accessible

## Installation

### Option 1: HACS (Recommended)

1. Open HACS → Add-ons → Repositories
2. Click **Add Repository**
3. Paste: `https://github.com/<your-username>/a3-odbd2-soc`
4. Click **Add**
5. Find **A3 PHEV HV SOC** and click **Install**

### Option 2: Custom Repository

1. Go to: **Settings → Add-ons → Add-on Store → Repositories**
2. Click **Add Repository**
3. Paste: `https://github.com/<your-username>/a3-odbd2-soc`
4. Click **Add**
5. Find **A3 PHEV HV SOC** and click **Install**

## Configuration

After installation, go to the add-on settings:

| Option | Description | Default |
|--------|-------------|---------|
| `mac` | BLE adapter MAC address | `00:10:CC:4F:36:03` |
| `pid` | OBD PID to read | `5B` |
| `mqtt_host` | MQTT broker hostname | `core-mosquitto` |
| `mqtt_port` | MQTT broker port | `1883` |
| `mqtt_username` | MQTT username (empty = anonymous) | |
| `mqtt_password` | MQTT password (empty = anonymous) | |
| `net_usable_kwh` | Net usable HV battery capacity in kWh | `12.4` |

## MQTT Topics

| Topic | Description |
|-------|-------------|
| `car/a3/hv_soc` | HV battery SOC percentage |
| `car/a3/hv_soc/availability` | Online/offline status |
| `car/a3/hv_kwh` | Estimated available energy (kWh) |

## Home Assistant Sensors

Add these to `configuration.yaml`:

```yaml
mqtt:
  broker: core-mosquitto  # or your external broker

sensor:
  - name: "A3 PHEV HV SOC"
    unique_id: a3_phev_hv_soc
    state_topic: "car/a3/hv_soc"
    availability_topic: "car/a3/hv_soc/availability"
    payload_available: "online"
    payload_not_available: "offline"
    value_template: "{{ value | float }}"
    device_class: battery
    state_class: measurement
    unit_of_measurement: "%"

  - name: "A3 PHEV HV Available Energy"
    unique_id: a3_phev_hv_available_energy
    state_topic: "car/a3/hv_kwh"
    availability_topic: "car/a3/hv_soc/availability"
    payload_available: "online"
    payload_not_available: "offline"
    value_template: "{{ value | float }}"
    state_class: measurement
    unit_of_measurement: "kWh"
```

## Updating

1. Go to **Settings → Add-ons**
2. Click on **A3 PHEV HV SOC**
3. Click **Update** in the top-right corner

Or use HACS → Add-ons → Updates.

## Troubleshooting

### "Cannot connect to MQTT broker"
- Enable built-in Mosquitto: **Settings → System → Services → Mosquitto**
- Or configure external MQTT in the add-on settings

### BLE connection failures
- Verify BLE adapter MAC is correct
- Ensure car is awake (not in sleep mode)
- Check BLE adapter pairing in HAOS

### "No values returned"
- Verify OBD port is accessible
- Check that PID `0x5B` works with your adapter
- Look at add-on logs for detailed errors

## Development

### Build locally
```bash
docker build -t a3-odbd2-soc .
```

### Run locally
```bash
docker run -d --name a3-odbd2-soc \
  --network host \
  -e MQTT_HOST=your-broker \
  -e MQTT_USERNAME=your-username \
  -e MQTT_PASSWORD=your-password \
  -e MAC=your-ble-mac \
  a3-odbd2-soc
```

## License

MIT
