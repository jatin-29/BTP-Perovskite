# Questions about the public dataset (draft for the student to send)

These notes describe what was observed in `perovs_data_final.xlsx` from github.com/yjq829/perovskite_dataset
(repository commit b84ed4d, 13 Apr 2023; file sha256 5ee941331477bb6038696c98e570846c3411b2e4b7216df5a4e6da0fabf7f862),
compared with arXiv 2302.04896v1. They are observations, not conclusions. The student should check them before sending.

1. **Identical HSErel_SOC and HSE-PBE-SOC gaps.** In 41 of the 244 compounds that have both values, the HSErel_SOC band
   gap is exactly equal to the HSE-PBE-SOC band gap. 37 of them have only Ca/Sr/Ba on the B site, 4 have a mixed B site,
   and none has only Ge/Sn/Pb. Examples: CsCaBr3, KSrI3, RbBaCl3. Is this expected (for example, the same calculation
   used for both entries), or a copying step in the table?

2. **MASrI3: HSErel gap below the PBE gap.** MASrI3 has PBE 3.763 eV, HSErel 3.465 eV, HSErel_SOC 3.406 eV and
   HSE-PBE-SOC 4.572 eV. For every other pure Ca/Sr/Ba compound the HSErel gap is above the PBE gap. Could the
   HSErel calculation of MASrI3 have converged to a different structure?

3. **RbGeBr1.125Cl1.875: large difference between the HSE levels.** This compound has HSErel 4.311 eV and HSErel_SOC 4.280 eV,
   but HSE-PBE-SOC 1.336 eV. It has no row in the PBE sheet, although the HSE-PBE-SOC level is described as a
   calculation on PBE-relaxed structures. Similar pure compounds (RbGeCl3, RbGeBr3) have HSErel gaps below 3 eV.

4. **Number of compounds.** The paper (Table 1, p. 4) lists 495 PBE compounds. The PBE sheet of the public file has
   488 rows: 90 pure, 124 A-site mixed, 151 B-site mixed and 123 X-site mixed. Section 2.1 of the paper gives 90 / 126 / 151 / 127
   (sum 494). Two compounds (RbGeBr1.125Cl1.875, Rb0.375Cs0.625GeBr3) have HSE values but no PBE row. Which PBE
   entries are missing from the public file?

5. **Table SII.** The last four rows of Table SII (PBE gaps of screened compounds, p. 23 of the arXiv version) do not
   match the PBE sheet. CsSnI1.875Br1.125: table 2.47, file 0.547 eV. CsPbI0.75Br2.25: table 2.49, file 1.619 eV.
   CsPbBr2.625Cl0.375: table 1.98, file 1.780 eV. MAPbI2.25Br0.75: table 1.53, file 1.829 eV. The other 42 printed gaps in
   Tables SII and SIII agree with the file within 0.01 eV.

6. **"Bartel's Tolerance Factor" column.** The column holds (rA/rB)/ln(rA/rB) (it matches to about 1e-14), not the
   full tau of Eq. 11 in the paper. Is this intended?

7. **Ionic radii.** The radii behind the "Tolerance Factor" and "Octaehdral Factor" columns can be recovered exactly.
   They look like a mixture of Shannon values (K, Rb, Cs, Sr, Ba at coordination VIII; Pb, Ca and the halides at
   coordination VI; Ge as the coordination-VI crystal radius 0.87 Å; Sn 1.15 Å, which is not in the Shannon table;
   MA 2.17 Å and FA 2.53 Å). Which source was used?
