# SeisComP MeRT

Real-time energy magnitude (*Me*) module for SeisComP.

Joint effort of GFZ Sections 2.6 (Seismic Hazard and Risk Dynamics) and 2.4 (Seismology), implementing the methodology from:

> Di Giacomo, D., Grosser, H., Parolai, S., Bormann, P., and Wang, R. (2008),
> Rapid determination of *Me* for strong to great shallow earthquakes,
> *Geophys. Res. Lett.*, 35, L10308, doi:10.1029/2008GL033505

## What it does

`scmert` monitors incoming SeisComP event messages in real-time. For events above the configured magnitude threshold it:

1. Retrieves P-wave picks and waveforms from the record stream
2. Removes instrumental response and applies a bandpass filter (0.02–1.0 Hz)
3. Computes the radiated seismic energy *E_S* per station using spectral amplitude decay corrections (pre-computed from AK135Q Green's functions)
4. Derives per-station *Me* via *Me = 2/3 (log₁₀ E_S − 4.4)*
5. Combines station values into a network *Me* using a trimmed mean
6. Publishes the result back to the SeisComP messaging/database as a `Magnitude` of type `Me`

Applicable for M > 5.8 shallow earthquakes at teleseismic distances (20°–98°).

## Integration into SeisComP

This module cannot be built standalone. It must be integrated into the SeisComP build environment:

```bash
git clone [host]/seiscomp.git
cd seiscomp/src/extras
git clone git@github.com:comoglu/scmert.git mert
```

Then follow the standard SeisComP build instructions from the `seiscomp` repository.

## Configuration

Parameters are set in the SeisComP module configuration (e.g. `~/.seiscomp/scmert.cfg`):

| Parameter | Default | Description |
|-----------|---------|-------------|
| `magnitudeThreshold` | `5.8` | Minimum preferred magnitude to trigger *Me* computation |
| `waveformCacheSize` | `500` | LRU cache size for waveform data |
| `inventoryCacheSize` | `500` | LRU cache size for station inventory |
| `yamlConfig` | *(built-in)* | Path to a custom `process.yaml` (overrides the built-in defaults) |

Advanced processing parameters (filter corners, SNR threshold, window durations, frequency–distance correction tables) are in `libs/python/mert/process.yaml`.

## Logging

At startup, configured parameter values are logged. During processing, each event produces per-station lines and a single summary line:

```
EVT 2024abcd: Me=6.54 +/-0.18 from 8/12 P stations
```

Stations skipped due to low SNR, distance out of range, saturation, or waveform gaps are reported inline with a reason.

## References

- Di Giacomo et al. (2008) — core methodology and frequency–distance correction tables (AK135Q)
- Bormann et al. (2002) — *Me = 2/3 (log₁₀ E_S − 4.4)* relationship
- Bormann & Wylegalla (2005) — cumulative body-wave magnitude concept
