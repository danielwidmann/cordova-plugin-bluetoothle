# Periodic Advertising API Implementation

## Summary
Added support for the Android Periodic Advertiser API (available from API level 26 / Android 8.0 Oreo) while maintaining backward compatibility with the legacy advertising API for older devices.

## Changes Made

### 1. Added New Imports
- `AdvertisingSet`
- `AdvertisingSetCallback`
- `AdvertisingSetParameters`
- `PeriodicAdvertisingParameters`

### 2. Added Class-Level Variables
- `currentAdvertisingSet`: Stores the current advertising set for API 26+
- `advertisingSetCallback`: Callback handler for the new advertising API

### 3. Enhanced `createAdvertiseCallback()` Method
- Added `AdvertisingSetCallback` implementation for API 26+
- Handles advertising start/stop events
- Provides comprehensive error handling with proper status codes

### 4. Updated `startAdvertisingAction()` Method
The method now includes conditional logic based on Android API level:

#### For API 26+ (Android O and above):
- Uses `AdvertisingSetParameters.Builder()` instead of `AdvertiseSettings.Builder()`
- Sets `setLegacyMode(true)` for compatibility
- **Implements interval-based control** instead of mode-based:
  - **Low Latency**: 80 units (50ms interval) - as requested
  - **Balanced**: 400 units (250ms interval)
  - **Low Power**: 1600 units (1000ms interval)
  - Note: Interval units are in 0.625ms increments
- Supports optional periodic advertising with `enablePeriodic` parameter
- Uses `startAdvertisingSet()` API

#### For API < 26 (Legacy):
- Maintains original implementation using `AdvertiseSettings.Builder()`
- Uses traditional advertising modes (LOW_LATENCY, BALANCED, LOW_POWER)
- Uses `startAdvertising()` API

### 5. Updated `stopAdvertisingAction()` Method
- Detects which API is in use
- For API 26+: Uses `stopAdvertisingSet()` with `advertisingSetCallback`
- For older APIs: Uses `stopAdvertising()` with `advertiseCallback`
- Properly cleans up `currentAdvertisingSet` reference

## Key Features

### Interval Control
The new API provides precise control over advertising intervals:
- **Low Latency mode**: 80 × 0.625ms = **50ms interval** (as specified in requirements)
- More granular control compared to the legacy mode-based approach

### Backward Compatibility
- All existing functionality preserved for devices running Android < 8.0
- Seamless fallback to legacy API when new API is unavailable
- No breaking changes to existing API interface

### Optional Periodic Advertising
The implementation supports optional periodic advertising parameters:
```javascript
{
  "enablePeriodic": true,
  "periodicIncludeTxPower": true,
  "periodicInterval": 80  // Optional, defaults to advertising interval
}
```

## API Reference
- [AdvertisingSetParameters.Builder](https://developer.android.com/reference/android/bluetooth/le/AdvertisingSetParameters.Builder)
- [PeriodicAdvertisingParameters.Builder](https://developer.android.com/reference/android/bluetooth/le/PeriodicAdvertisingParameters.Builder)
- [AdvertisingSetCallback](https://developer.android.com/reference/android/bluetooth/le/AdvertisingSetCallback)

## Testing Recommendations
1. Test on devices with API 26+ to verify new advertising behavior
2. Test on devices with API < 26 to ensure legacy behavior works
3. Verify low latency mode uses 50ms intervals on supported devices
4. Test periodic advertising if enabled
5. Verify proper cleanup when stopping advertising

## Notes
- The implementation uses `setLegacyMode(true)` to ensure compatibility with devices scanning using the legacy API
- TX power levels are mapped appropriately between old and new APIs
- The timeout parameter is not used in the new API (handled differently in AdvertisingSet)
