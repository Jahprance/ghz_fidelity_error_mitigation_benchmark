# GHZ-State Fidelity Benchmarking and Quantum Error Mitigation

**Author:** Praneshraj Tiruppur Nagarajan Dhyaneswar  
**Tools:** Python, Qiskit, Qiskit Aer 
**Scope:** Controlled simulation and FakeManila compilation. No quantum
hardware execution is included.

## Objective

This project investigates how reliably GHZ-state fidelity can be estimated
from noisy finite-shot measurements, and whether error-mitigation methods
improve the estimate enough to justify their additional sampling cost.

A higher reported fidelity is not automatically a better estimate. I compare
each method with a defined reference and include the target-measurement and
calibration shots required by that method.

## References used

The final comparison uses two references:

- **Prepared-state reference:** exact fidelity of the noisy scale-1 state,
  calculated from the simulated density matrix before analysis rotations and
  readout.
- **Ideal reference:** fidelity 1.0 for the intended noiseless GHZ-plus
  state.

The references answer different questions. A method can move an estimate
closer to the ideal target while moving it farther from the noisy state that
was actually prepared.

## Workflow

The notebook builds the benchmark step by step:

1. Prepare Bell and GHZ-plus states from 3 to 5 qubits and verify ideal populations
   and target fidelity.
2. Compare GHZ+, GHZ- and balanced classical mixture to show why
   computational-basis populations alone cannot fully describe GHZ fidelity.
3. Validate measurement rotations and reconstruct fidelity using endpoint
   populations and parity measurements that recover GHZ coherence.
4. Use density-matrix simulation to separate preparation noise from bias
   introduced by noisy analysis rotations.
5. Run finite-shot pilots under ideal, readout-only, depolarizing-only and
   combined-noise conditions.
6. Calibrate local readout assignment matrices and correct measured
   distributions without clipping negative quasi-probabilities.
7. Fold only the GHZ preparation circuit at scales 1, 3, 5 verify that
   the ideal output is unchanged and fit linear ZNE estimates.
8. Compare unmitigated, readout mitigation, ZNE only and readout mitigation
   plus ZNE over 20 matched repetitions for each width.
9. Compare bias, MAE, RMSE, estimate variation, paired error changes,
   calibration behaviour and total-shot cost.
10. Transpile GHZ circuits on FakeManila across optimization levels and
    forced physical layouts.

The main benchmark contains 80 matched width-repeat experiments and 320
method-result rows. Readout calibration is repeated for every run, and its
cost is included in the total-shot comparison.

## Main findings

For the noisy prepared-state reference, readout mitigation gave the lowest
MAE for every tested width from 2 to 5 qubits. Its MAE ranged from 0.013671
to 0.018029, and it improved absolute error over the unmitigated result in
every matched run.

For GHZ(5), the unmitigated prepared-reference MAE was 0.250655. Readout
mitigation reduced it to 0.015910, a 93.65% reduction. This used 4,096 total
shots, compared with 3,072 shots for the unmitigated method because it
included 1,024 calibration shots.

For the ideal reference, readout mitigation had the lowest observed MAE at
GHZ(2). At GHZ(3) through GHZ(5) readout mitigation plus ZNE had the lowest
observed MAE, but required substantially more shots. At GHZ(5), its
ideal-reference MAE was 0.019688 using 10,240 shots. Its prepared-reference
MAE was 0.044976, which was higher than readout mitigation alone.

For the FakeManila compilation check, optimization levels 0 to 3 produced
the same CX counts and circuit depths for the standard GHZ chains. In a
controlled GHZ(3) layout comparison, changing the initial layout from
`[0, 1, 2]` to `[0, 2, 4]` increased physical CX count from 2 to 8 and
compiled circuit depth from 5 to 11 because routing was required.

## Accuracy and shot cost

Each point in the final accuracy-cost figure is the mean MAE from 20 matched
repetitions for one method and GHZ width. Lines connect widths from 2 to 5
qubits; they are not fixed-shot-budget curves.

Shot counts include target measurements and calibration where applicable.
ZNE uses measurements at scales 1, 3, and 5, so its total shot cost is
higher. Error bars are not shown; the figure shows observed mean MAE and
does not demonstrate statistical significance.

## Repository structure

```text
.
├── figures/       Generated PNG figures
├── notebooks/     Final executed Colab notebook
├── results/       CSV results and experiment configuration
├── README.md      Project overview
└── requirements.txt
```

## Reproduction

1. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

2. Open the notebook in `notebooks/`.
3. Run cells in order.

The notebook records package versions, configuration, simulator seeds,
calibration records, raw counts, and saved result tables.

## Scope and limitations

These results apply to the selected GHZ circuits, controlled noise model,
calibration approach, and shot budgets. The project uses Aer simulation and
FakeManila transpilation.. it does not execute on real quantum hardware.

Readout errors are independent across qubits. ZNE folds only the GHZ
preparation circuit at scales 1, 3, and 5. Analysis rotations and readout are
not folded, so the ZNE intercept should not be treated as a completely
noise-free measurement result.

The methods use the same shots per target measurement setting, but different
total shot budgets. The results describe observed average behaviour over 20
repeats and do not establish statistical significance.
