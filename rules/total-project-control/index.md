# Total Project Control

total_project_control:

- total_project_control_dipp:
  - help: Total Project Control (TPC) Devaux's Index of Project Performance (DIPP)
  - type: decimal
- total_project_control_dipp_progress_index_ratio:
  - help: Total Project Control (TPC) Devaux's Index of Project Performance (DIPP) Progress Index (PI)
  - type: decimal
  - sql: total_project_control_dipp_progress_index_ratio GENERATED ALWAYS AS (total_project_control_dipp_progress_index_numerator / total_project_control_dipp_progress_index_denominator)
- total_project_control_dipp_progress_index_numerator:
  - help: Total Project Control (TPC) Devaux's Index of Project Performance (DIPP) Progress Index (PI) Numerator
  - type: decimal
- total_project_control_dipp_progress_index_denominator:
  - help: Total Project Control (TPC) Devaux's Index of Project Performance (DIPP) Progress Index (PI) Denominator
  - type: decimal
- total_project_control_expected_monetary_value:
  - help: Total Project Control (TPC) Expected Monetary Value (EMV)
  - type: decimal
- total_project_control_cost_estimate_to_complete:
  - help: Total Project Control (TPC) Cost Estimate to Complete (CEC)
  - type: decimal
