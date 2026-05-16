# Convolver Config Generator

**[Open the tool](https://kmarkley.github.io/ConvolverConfigGenerator/)**

A browser-based tool for building and editing configuration files for the [Convolver](https://convolver.sourceforge.net/) convolution engine (v3.0+ format).

## Usage

Open `index.html` directly in a browser — no build step, no server required.

## Features

### Global Parameters
- **Sample Rate** — written to the config header
- **Input / Output Channels** — sets the matrix dimensions (1–16 each)
- **Atten Mode** — switch between entering attenuation as **dB** (≤ 0) or **Scale Factor** (0 to < 1.0); existing values are converted when the mode changes

### Channel Labels & Delays
Each input and output channel can be given a short label (used in matrix headers and config comments). Per-channel delay values (ms) are shown with the label of the channel they belong to, and update live as labels change.

### Filter Matrix
The main grid represents every input → output routing path. Each cell contains:

| Field | Notes |
|---|---|
| Enable checkbox | Hidden fields collapse when disabled |
| Filter File | Click (or focus + Enter/Space) to edit |
| Filter Channel | Integer ≥ 0 — the channel index within the IR file |
| Input Atten | ≤ 0 dB or 0–0.9999 scale depending on Atten Mode |
| Output Atten | Same constraints as Input Atten |

A **green** left stripe marks enabled cells with all filters configured. A **red** stripe marks enabled cells with one or more filters missing a file. 

Numbered badges in the top-right corner show export path assignments — see [Path Badges](#path-badges) below.

### Path Badges

Every enabled cell with at least one filter file set displays one or more numbered badges in its top-right corner. 

Each badge shows the cfg export path number that filter will occupy. 

#### Filter Path Grouping

The Convolver format allows multiple input and output channels to be listed on a single filter path, mixing inputs before convolution and distributing the result to multiple outputs. This is more efficient than running separate convolutions for the same IR file.

Path badges are color-coded by path grouping: cells whose filters are combined into the same multi-token path share a color; filters that must be emitted as separate paths each get a distinct color; solo paths (no grouping possible) show a grey badge.

The tool automatically groups compatible cells into combined paths on export. Cells can share a filter path only when:

- They reference the **same filter file and channel**
- Their enabled (inCh, outCh) pairs form a **complete rectangle** — every combination of the grouped inputs and outputs must be active
- Each input channel has a **consistent input attenuation** across all outputs in the group, and each output channel has a **consistent output attenuation** across all inputs

When cells can't be fully combined (e.g. a diagonal routing where ch0→ch6 and ch1→ch7 use the same IR but ch0→ch7 and ch1→ch6 are not enabled), the tool automatically splits them into separate paths. The path badges make this visible before you export.

On import, multi-token paths are expanded back into individual matrix cells correctly.

#### Stacked Filters

Each cell can hold **multiple filter definitions** for the same input → output path. This is useful when a signal needs to pass through independent convolutions that Convolver sums together — for example, an independent high-pass filter paired with a shared low-pass filter.

The `+` and `×` controls to the left of the path badges add or remove filters on the cell. When a cell has more than one filter, clicking a badge switches which filter's fields are shown in the cell — the active filter is indicated by a white highlight ring.

Each stacked filter has its own file, channel, and attenuation values, and becomes a separate 4-line block in the exported cfg file (all referencing the same input and output channels). On import, multiple blocks targeting the same channel pair are automatically restored as stacked filters in the same cell.

### Config Output
The generated `.cfg` text updates live. From the output toolbar:

- **Import .cfg** — load an existing config file to populate the editor; channel labels, delays, and all filter paths are restored. Attenuation values are converted to dB on import.
- **Copy** — copies the generated text to the clipboard
- **Filename field** — sets the download filename; `.cfg` is appended silently if no extension is given
- **Download** — saves the file using the native Save As dialog (`showSaveFilePicker`) when available, falling back to a direct browser download

## Config File Format

```
<sample_rate> <num_inputs> <num_outputs> <output_mask_hex>
<input_delay_ms> ...           # one value per input channel
<output_delay_ms> ...          # one value per output channel

# repeated for each active filter path:
<filter_filename>
<filter_channel>               # 0-based index within the IR file
<in_ch>.<in_scale> [...]       # one or more space-separated input tokens
<out_ch>.<out_scale> [...]     # one or more space-separated output tokens
```

Scale notation uses a dot as a **separator**, not a decimal point — `1.5` means channel 1, scale 0.5. Multiple tokens on the input or output line cause Convolver to mix those channels before (or distribute after) convolution, which is more efficient than separate filter paths for the same IR.

## Browser Compatibility

Works in any modern browser. The native Save As dialog requires Chrome 86+ or Edge 86+; other browsers fall back to a standard download using the filename field value.

## License

This project is licensed under the [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.html).
