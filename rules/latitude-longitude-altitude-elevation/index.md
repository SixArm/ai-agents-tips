# latitude, longitude, altitude, elevation

Geography geolocation source code naming convention: use the decimal unit name.

- latitude -> `latitude_as_decimal_degrees`
- longitude -> `longitude_as_decimal_degrees`
- altitude -> `altitude_as_decimal_metres`
- elevation -> `elevation_as_decimal_metres`

If a variable or field needs a type, then always use a decimal type, never use a float type.

## Exclude

Exclude Cascading Style Sheet (CSS) files that mention elevation when the word is design-system shadow depth, not geography.
