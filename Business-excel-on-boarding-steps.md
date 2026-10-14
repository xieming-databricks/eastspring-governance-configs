Prerequisite
- reference/esg.yaml
- excels/esg/access-matrix-esg_workshop-v3-dynamic-views.xlsx
- Python 3.10+ (3.13 recommended); run every command from the root of this repo

<table>
<tr><th></th><th>macOS / Linux (bash / zsh)</th><th>Windows (PowerShell)</th></tr>

<tr><td><b>VirtualEnv</b></td>
<td><pre>
python3 -m venv .venv
source .venv/bin/activate
</pre></td>
<td><pre>
python -m venv .venv
.\.venv\Scripts\Activate.ps1
</pre>
</td></tr>

<tr><td><b>Install Wheel</b></td>
<td><pre>
pip install /path/to/wheel/eastspring_uc_governance-0.2.0-py3-none-any.whl
</pre></td>
<td><pre>
pip install &lt;path-to&gt;\dist\eastspring_uc_governance-0.2.0-py3-none-any.whl
</pre></td></tr>

<tr><td><b>Generate yaml from excel</b></td>
<td><pre>
ucgov-generate excels/esg/access-matrix-esg_workshop-v3-dynamic-views.xlsx config/esg --check

ucgov-generate excels/esg/access-matrix-esg_workshop-v3-dynamic-views.xlsx config/esg
</pre></td>
<td><pre>
ucgov-generate excels\esg\access-matrix-esg_workshop-v3-dynamic-views.xlsx config\esg --check

ucgov-generate excels\esg\access-matrix-esg_workshop-v3-dynamic-views.xlsx config\esg
</pre></td></tr>

<tr><td><b>Validate</b></td>
<td><pre>
ucgov-validate --reference reference/esg.yaml config/esg --verbose
</pre></td>
<td><pre>
ucgov-validate --reference reference\esg.yaml config\esg --verbose
</pre></td></tr>
</table>

Git Commit

Trigger onboarding-pipeline
