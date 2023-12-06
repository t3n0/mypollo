# Quantum Espresso workflows

A table with some of the possible workflows of Quantum Espresso.
Each column represents a property that we want to calculate.
For any given column, the entries are the executable tools (e.g. `pw.x`, `ph.x`, `dos.x`, etc.) needed to be run to execute that specific workflow.
Each entry shows a brief description of the executable and links to an example.
The top entry of any workflow is a link to a zip file that can be download and run.

|            | DOS                     | Bands                                        | proj Bands | spin Bands | kres DOS | Wannier90 |
|------------|-------------------------|----------------------------------------------|-------|--------|--------|------|
|`pw.x`      | (1) scf<br><br>(2) nscf | (1) scf <br><br> (2) nscf <br><br> (3) bands | scf <br><br> nscf <br><br> bands | scf <br><br> nscf <br><br> bands | scf <br><br> nscf <br><br> bands | |
|`bands.x`   |                         | (4.1) default <br><br> (4.2) non-coll        | default | spin | default | |
|`projwfc.x` | (3) default             | (6.1) proj <br><br> (6.2) k-resolve          | proj | spin | kres | |
|`dos.x`     | (4) default             |                                              | | | | |
|`plotband.x`|                         | (5.1) default <br><br> (5.2) non-coll <br><br> (7) proj | proj | spin | | |

