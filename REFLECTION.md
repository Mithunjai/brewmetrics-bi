# Project Reflection


GitHub Copilot was useful during the development of the BrewMetrics DAX measures by providing initial patterns for time-based calculations, cumulative totals, and ranking. For example, it suggested approaches using `DATEADD` and `DIVIDE` for month-over-month growth, `ALLSELECTED` for cumulative sales, and `RANKX` for product ranking. These suggestions gave me a starting point that I could compare against the semantic model and the requirements of the analysis.

My main contribution was verification and validation rather than changing Copilot-generated formulas. I checked that the time-based calculation used `Dim_Date[date]`, that product ranking used `Dim_Product[item]`, and that the documented measures matched the actual semantic model. I did not find a formula that required correction, so I have not claimed a correction that did not occur. I also identified a mismatch between a commit message referring to an average-sale measure and the measure actually present in the model, which showed the importance of checking the repository contents rather than relying only on commit titles.

Git changed my workflow from treating the Power BI report as a single finished file to developing it through reviewable stages. The commit history separates the model, DAX development, dashboard work, and documentation, making changes easier to trace and compare. This also made me more conscious of validating both the actual files and the documentation throughout the development process.
