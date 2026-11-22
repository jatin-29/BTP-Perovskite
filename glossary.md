# Glossary (plain language)

**PBE.** A common, cheap approximation in density functional theory (DFT), a generalized-gradient functional. It usually
usually gives band gaps that are too small. In this dataset, "PBE" also means the crystal structure
was relaxed with PBE.

**HSE (HSE06).** A "hybrid" functional that mixes part of the exact (Hartree-Fock) exchange into PBE. It opens the band
gap and is usually closer to experiment, but it costs much more to compute. In the dataset, "HSErel" means the structure
was also relaxed with HSE, and "HSE-PBE-SOC" means a single HSE+SOC calculation on the PBE-relaxed structure.

**SOC (spin-orbit coupling).** The interaction between an electron's spin and its orbital motion. It is strong for
heavy atoms such as Pb and splits and lowers the conduction band, so including SOC reduces the gap. The "SOC shift"
in this project is HSErel_SOC minus HSErel.

**Tolerance factor (Goldschmidt t).** t = (rA + rX) / (sqrt(2) (rB + rX)), built from the ionic radii of the A, B and X
ions. It measures how well the A cation fits the hole between the BX6 octahedra. Values near 1 suit a cubic perovskite;
values far from 1 favour distorted or non-perovskite structures.

**Bartel tau.** A newer tolerance factor (Bartel et al., 2019): tau = rX/rB - nA (nA - (rA/rB)/ln(rA/rB)), where nA is
the oxidation state of A. In that paper tau < 4.18 is used to predict that a compound forms a perovskite.

**Octahedral factor (mu).** mu = rB / rX. It measures whether the B cation is large enough to sit stably in an octahedron
of six X anions.

**Lone pair (ns2).** Ge2+, Sn2+ and Pb2+ (Group 14) keep two outer s electrons (4s2, 5s2, 6s2) that do not take part in
bonding. These s states mix with the halide p states to form the top of the valence band. Ca2+, Sr2+ and Ba2+ (Group 2)
have no such electrons (noble-gas-like), which is one reason their gaps are much larger.

**Grouped cross-validation.** Cross-validation where whole groups of related samples (here, compounds with the same
main A, B and X elements) are kept together in either the training or the test fold. This stops the model from being
tested on near-copies of its training compounds. It gives a more honest estimate than a random split.

**Leakage.** Information about the test set getting into training. Examples: a test compound has a close sibling in
training, or a hyperparameter is tuned on the same folds that are reported. Leakage makes scores look better than they
would be on genuinely new data.

**Leave-one-element-out (clean).** All compounds whose B site is only the held-out element form the test set. The
training set contains no compound with any amount of that element. It asks whether the model works for an element it
has never seen.

**MAE (mean absolute error).** The average of |predicted - true|, here in eV. An MAE of 0.3 eV means the predictions are
off by 0.3 eV on average.

**R2 (coefficient of determination).** 1 minus (sum of squared errors / total variance of the true values). R2 = 1 is
perfect, R2 = 0 is no better than always predicting the mean, and R2 < 0 is worse than predicting the mean.

**Mean baseline.** A "model" that always predicts the mean gap of the training set. Any useful model must beat it.

**Bootstrap.** Repeatedly resample the data with replacement (here 500 times), refit or re-evaluate each time, and look
at the spread of the results. The middle 95 percent of that spread is used as an approximate 95 percent interval.
A "group bootstrap" resamples whole groups instead of single rows.

**Variance explained (eta squared).** The fraction of the spread in gaps that is removed when each compound is replaced
by the mean of its category (for example, its halide). One-way fractions for different labels can overlap, so they do
not add up to 1.

**Bowing.** In a mixed compound such as A B (I_x Br_(1-x))3, the gap often does not follow a straight line between the
two pure end members. The deviation from that line is called bowing.

**Deviation of cubicity.** Dataset columns that measure how far the relaxed cell is from a perfect cube (axis lengths
and angles). They come from the PBE-relaxed structures.
