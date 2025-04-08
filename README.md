# Interactive Multivariable qPCR Simulator

**By Filippo D. Galbo**

## Introduction

This project simulates a multiplex quantitative real-time PCR (qPCR) reaction within a single, standalone HTML file using the p5.js library. It provides an interactive and educational platform to visualize how various experimental parameters influence amplification curves (both copy number and fluorescence) and the resulting quantification cycle (Ct) values. The simulation incorporates several factors to enhance realism, including sigmoidal growth, stochastic effects in early cycles, and penalties based on temperature, inhibitors, and primer/probe characteristics.

## Features

*   **Multiplex Simulation:** Simulates up to 4 distinct targets simultaneously, each with a unique color.
*   **Dual Plots:**
    *   **Top Plot:** Cycle Number vs. Copy Number (Log Scale Y-axis).
    *   **Bottom Plot:** Cycle Number vs. Fluorescence (Log Scale Y-axis).
*   **Interactive Ct Threshold:**
    *   Adjust the fluorescence threshold using a slider *before or after* the run.
    *   Drag the threshold line directly on the fluorescence plot after the run to dynamically recalculate Ct values.
*   **Editable Target Names:** Customize the name for each target being simulated.
*   **Adjustable Parameters:** Fine-tune numerous settings via sliders and inputs:
    *   **General:** Number of Targets, Max Cycles, Fluorescence Threshold (Ct), Max Fluorescence (Saturation).
    *   **Temperature:** Denaturation, Annealing, and Extension temperatures (°C), with deviations from ideal values impacting efficiency.
    *   **Inhibitors:** Select common inhibitor types (Heparin, Humic Acid, etc.) and concentration levels, applying a global penalty.
    *   **Target-Specific:**
        *   Initial Template Copies (log10 scale).
        *   Base Amplification Efficiency (%).
        *   Primer/Probe Length (bp).
        *   Sequence Match (%).
*   **Realistic Modeling:**
    *   **Sigmoidal Growth:** Simulates reaction plateau due to reagent limitations using a carrying capacity model based on copy number.
    *   **Stochasticity:** Incorporates binomial probability for amplification success in early cycles (below ~1000 copies) to mimic real-world variability.
    *   **Efficiency Penalties:** Calculates effective efficiency based on temperature deviations, primer/probe length/match issues, and inhibitor presence.
    *   **Baseline Simulation:** Adds subtle noise and drift to the initial fluorescence cycles.
    *   **Optional Baseline Subtraction:** A checkbox allows subtracting a calculated linear baseline (from cycles 3-10) from the fluorescence plot for visualization.
*   **Real-Time Visualization:** Observe the amplification curves develop cycle by cycle during the simulation run (~5 seconds duration).
*   **User Interface:** Intuitive controls panel with sliders, inputs, buttons (Start, Stop, Reset), and real-time feedback on status, cycle number, efficiency, and Ct values.
*   **Standalone:** All necessary HTML, CSS, and JavaScript (p5.js included via CDN) are contained within a single `.html` file.
*   **Educational:** Designed to illustrate key qPCR concepts like exponential amplification, efficiency impact, Ct values, plateau effects, inhibition, and sources of variability.

## How to Use

1.  **Save:** Save the entire code block provided as an HTML file (e.g., `qpcr_simulator.html`).
2.  **Open:** Open the saved `qpcr_simulator.html` file in a modern web browser (Chrome, Firefox, Edge, Safari recommended).
3.  **Adjust Settings:** Use the controls panel on the left to configure the desired simulation parameters *before* starting.
4.  **Run:** Click the "Start" button. The simulation will run for approximately 5 seconds, plotting data cycle by cycle. Controls will be disabled during the run.
5.  **Observe:** Watch the Copy Number and Fluorescence plots update. Note the calculated Max Efficiency and Ct Values in the results area below the canvas container.
6.  **Interact Post-Run:**
    *   Use the "Fluorescence Threshold (Ct)" slider to change the threshold and see Ct values update.
    *   Click and drag the green dashed line on the *bottom* (Fluorescence) plot to visually adjust the threshold.
    *   Toggle the "Subtract Calculated Baseline" checkbox to see its effect on the fluorescence plot.
7.  **Stop/Reset:**
    *   Click "Stop" to halt the simulation mid-run.
    *   Click "Reset" to clear the plots and results, re-enable all controls, and allow for a new simulation setup.

## Controls Explained

*   **General Settings:**
    *   `Number of Targets`: How many different DNA sequences to simulate (1-4).
    *   `Max Cycles`: The total number of amplification cycles to simulate.
    *   `Fluorescence Threshold (Ct)`: The fluorescence level used to determine the Ct value. Adjustable post-run.
    *   `Max Fluorescence (Saturation)`: The maximum fluorescence signal achievable (probe saturation/detection limit). Affects the plateau of the fluorescence curve.
    *   `Subtract Calculated Baseline`: Toggles visualization of baseline subtraction on the fluorescence plot.
*   **Cycling Temperatures:**
    *   Set Denaturation, Annealing, and Extension temperatures.
    *   Deviations from the ideal (95°C, 60°C, 72°C) reduce the calculated maximum efficiency via penalty factors.
*   **Inhibitor Settings:**
    *   `Inhibitor Type`: Select from common PCR inhibitors or "None".
    *   `Inhibitor Level`: An abstract level representing concentration/effect. Inhibitors apply a global penalty to efficiency.
*   **Target-Specific Controls (Per Target):**
    *   `Target Name`: Click to edit the default name (e.g., "Target 1").
    *   `Initial Copies (log10)`: Set the starting number of DNA copies on a logarithmic scale (e.g., 3.0 = 10^3 = 1000 copies).
    *   `Base Amplification Eff (%)`: The theoretical maximum efficiency (50-100%) for this target under ideal conditions *before* penalties.
    *   `Primer/Probe Length (bp)`: Lengths outside an optimal range (e.g., 18-30bp) incur a penalty.
    *   `Sequence Match (%)`: Lower match percentage significantly reduces binding and incurs a sharp efficiency penalty.
    *   *(Displays)*: Shows the calculated `Primer Penalty Factor` and final `Max Eff` for this target after all penalties are applied.
*   **Simulation Control:**
    *   `Start`: Begins the simulation with current settings.
    *   `Stop`: Halts an ongoing simulation.
    *   `Reset`: Clears results and resets the simulation to an idle state, ready for new settings.
    *   `Status`: Shows whether the simulation is Idle, Running, Stopped, or Finished.

## Technical Details

*   **Framework:** Built using p5.js for drawing and interactivity.
*   **Model:**
    *   Cycle-by-cycle calculation.
    *   **Efficiency:** A base efficiency is set per target, then modulated by penalty factors derived from temperature, primer length/match, and inhibitors to get a `maxEff`.
    *   **Sigmoid Growth:** The efficiency used in each cycle (`cycleEfficiency`) is further reduced based on product accumulation relative to a maximum carrying capacity (`maxPossibleCopies`), simulating reagent depletion/product inhibition: `cycleEfficiency = maxEff * (1 - prevCopyNum / maxPossibleCopies)`.
    *   **Stochasticity:** For `prevCopyNum < 1000`, amplification follows a binomial distribution (`binomialRandom(prevCopyNum, cycleEfficiency)`) to model random chance in early cycles. Above the threshold, it's deterministic.
    *   **Fluorescence:** Increases proportionally to current fluorescence and `cycleEfficiency`, capped by `saturationLevel`. Baseline noise and drift are added in early cycles.
    *   **Ct Calculation:** Determined by linear interpolation when the *raw* fluorescence signal crosses the `fluorescenceThreshold`. Recalculated dynamically if the threshold is changed post-run.

## Limitations

*   **Simplified Kinetics:** Does not use full mass-action kinetics or ODEs. Efficiency penalties are heuristic approximations.
*   **Abstract Parameters:** Inhibitor levels, penalty factors, and carrying capacity are abstract representations, not directly tied to specific molar concentrations or rate constants.
*   **No Primer-Dimers/Non-Specific Products:** Does not explicitly model the formation or impact of these common artifacts.
*   **No Explicit Reagent Tracking:** dNTP/primer/polymerase concentrations are not tracked explicitly; their depletion is implicitly modeled via the sigmoid function.
*   **No Melt Curve Analysis:** Does not simulate post-PCR melt curves.
*   **Fluorescence Model:** Assumes fluorescence is reasonably proportional to product amount after baseline, capped by saturation (a simplification, especially for intercalating dyes like SYBR Green).

## Future Ideas

*   Implement explicit primer-dimer competition model.
*   Add more specific mechanisms for different inhibitor types.
*   Allow export of simulated data (CSV/JSON).
*   Add interactive tooltips explaining graph features or parameters.
*   Simulate melt curve analysis based on target/PD properties.
*   Introduce options for simulating technical replicates with variations.
*   Develop a 96-well plate UI for setting up multiple simulations.
