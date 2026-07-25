# EXIF Metadata & GPS Coordinate Extraction Service

## Overview

The EXIF extraction service parses EXIF tags from uploaded image buffers to verify capture timestamp, camera model, and GPS coordinates. It provides a robust, type-safe solution for extracting and validating image metadata.

## Features

- **GPS Coordinate Extraction**: Extract latitude, longitude, altitude, and accuracy from EXIF GPS data
- **Camera Information**: Extract make, model, software, lens details, and shooting parameters
- **Timestamp Verification**: Parse and validate capture timestamps with timezone support
- **Validation**: Built-in validation for coordinates, timestamps, and camera models
- **Configurable**: Flexible configuration for extraction options and validation rules
- **Error Handling**: Comprehensive error handling with detailed error messages
- **Type-Safe**: Full TypeScript support with strict type definitions

## Installation

The service uses the `exifr` library which is already included in the project dependencies:

```bash
# Already installed
pnpm install exifr
```

## Usage

### Basic Usage

```typescript
import { extractEXIFMetadata } from '@/lib/services/exif-extraction';

const imageBuffer = Buffer.from(imageData);
const result = await extractEXIFMetadata(imageBuffer, 'image/jpeg');

if (result.success) {
  console.log('GPS:', result.metadata?.gps);
  console.log('Camera:', result.metadata?.camera);
  console.log('Timestamp:', result.metadata?.timestamp);
} else {
  console.error('Error:', result.error);
}
```

### Advanced Usage with Service Instance

```typescript
import { EXIFExtractionService } from '@/lib/services/exif-extraction';

const service = new EXIFExtractionService({
  extractGPS: true,
  extractCamera: true,
  extractTimestamp: true,
  validateCoordinates: true,
  maxFileSize: 10 * 1024 * 1024, // 10MB
  allowedMimeTypes: ['image/jpeg', 'image/webp'],
});

const result = await service.extractFromBuffer(imageBuffer, 'image/jpeg');
```

### Verification Methods

```typescript
// Verify timestamp is within acceptable range
const timestampCheck = service.verifyTimestamp(result.metadata!, {
  maxAge: 30 * 24 * 60 * 60 * 1000, // 30 days
  minDate: new Date('2024-01-01'),
  maxDate: new Date(),
});

// Verify camera model matches expected value
const cameraCheck = service.verifyCameraModel(result.metadata!, 'Canon EOS R5');

// Verify GPS coordinates are within expected region
const locationCheck = service.verifyGPSLocation(result.metadata!, {
  minLat: 30,
  maxLat: 40,
  minLon: -130,
  maxLon: -110,
});
```

## Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `extractGPS` | boolean | true | Extract GPS coordinates from EXIF data |
| `extractCamera` | boolean | true | Extract camera information |
| `extractTimestamp` | boolean | true | Extract timestamp information |
| `extractImageDimensions` | boolean | true | Extract image width and height |
| `validateCoordinates` | boolean | true | Validate GPS coordinates |
| `maxFileSize` | number | 52428800 | Maximum file size in bytes (50MB) |
| `allowedMimeTypes` | string[] | ['image/jpeg', 'image/jpg', 'image/tiff', 'image/webp'] | Allowed MIME types |

## Metadata Structure

### GPSCoordinates
```typescript
{
  latitude: number;      // Latitude in decimal degrees
  longitude: number;     // Longitude in decimal degrees
  altitude?: number;     // Altitude in meters
  accuracy?: number;     // GPS accuracy in meters
}
```

### CameraInfo
```typescript
{
  make?: string;         // Camera manufacturer
  model?: string;        // Camera model
  software?: string;     // Software/firmware version
  lensModel?: string;    // Lens model
  focalLength?: number;  // Focal length in mm
  iso?: number;          // ISO sensitivity
  aperture?: number;     // Aperture f-number
  shutterSpeed?: string; // Shutter speed (e.g., "1/250s")
}
```

### TimestampInfo
```typescript
{
  dateTimeOriginal?: Date;    // Original capture date/time
  dateTimeDigitized?: Date;    // Digitization date/time
  dateTime?: Date;            // Modification date/time
  offsetTime?: string;        // Timezone offset
  offsetTimeOriginal?: string; // Original timezone offset
}
```

## Validation

### GPS Coordinate Validation
- Latitude must be between -90 and 90
- Longitude must be between -180 and 180
- Altitude must be above Dead Sea elevation (-11,000m)

### Timestamp Validation
- Timestamp cannot be in the future
- Can enforce maximum age
- Can enforce date ranges

### Camera Model Verification
- Case-insensitive matching
- Supports partial model matching

## Error Handling

The service provides detailed error messages for:

- File size exceeded
- Invalid MIME type
- Missing EXIF data
- Parse errors
- Validation failures

```typescript
if (!result.success) {
  console.error(result.error);
  if (result.validationErrors) {
    result.validationErrors.forEach(err => console.error(err));
  }
}
```

## Security Considerations

- File size limits prevent memory exhaustion
- MIME type validation prevents processing of non-image files
- Coordinate validation prevents invalid GPS data
- Timestamp verification prevents future-dated or stale images

## Testing

Run tests with Vitest:

```bash
pnpm test lib/services/exif-extraction.test.ts
```

The test suite includes:
- EXIF extraction from valid buffers
- File size validation
- MIME type validation
- GPS coordinate extraction and validation
- Camera information extraction
- Timestamp verification
- Camera model verification
- GPS location verification
- Configuration options

## Performance Considerations

- Processing time depends on file size and EXIF data complexity
- Large files may take longer to parse
- Consider caching results for frequently accessed images
- Use streaming for very large files if needed

## Best Practices

1. **Always validate results**: Check `success` flag before using metadata
2. **Set appropriate file size limits**: Adjust based on your use case
3. **Verify timestamps**: Ensure images are recent enough for your application
4. **Validate GPS data**: Confirm coordinates are within expected regions
5. **Handle errors gracefully**: Provide user-friendly error messages
6. **Log extraction failures**: Monitor for suspicious patterns

## Troubleshooting

### No EXIF Data Found
- Ensure image contains EXIF metadata (some social media images strip EXIF)
- Check if image format supports EXIF (JPEG, TIFF, WebP)
- Verify image hasn't been processed by tools that remove metadata

### Invalid GPS Coordinates
- Check if device GPS was enabled when photo was taken
- Verify image wasn't edited with tools that modify GPS data
- Ensure coordinate validation rules aren't too strict

### Timestamp Issues
- EXIF dates use "YYYY:MM:DD HH:MM:SS" format
- Timezone information may be missing
- Some cameras may not set all timestamp fields

## License

MIT
