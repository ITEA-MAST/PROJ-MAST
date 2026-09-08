# PROJ-MAST - optimization tools
MAST-Managing Sustainability Trade-offs 
https://itea-mast.org/

# Existing repository links
- To be added

# Links to existing tools
- https://www.gecad.isep.ipp.pt/api/mast/clw_forecast_guide
- https://gecad.isep.ipp.pt/api/mast/clw_optimization_service/optimization-documentation
- https://github.com/PLEnergyDev/green-languages

# Links to tools under development
- https://gitlab.almende.org/research-projects/mast/crownstone-inside/documentation



## GECAD/CLW Generation | Consumption Forecast Service

This API orchestrates the energy generation/consumption forecast pipeline. It accepts a reference dataset and a forecast timeframe, executes processing scripts, and returns a consumption/generation forecast as a json response.

`BASE URL`  `http://gecad.isep.ipp.pt/api/mast/clw_forecast_api`

---

## Dataset Requirements

The uploaded `.xlsx` file must respect a specific structure specified bellow. The pipeline relies on specific column names to process time series and energy consumption data.

### 1a. Consumption Structure Example

| Device | TagId | TagName | Date | A | M | D | H | m | Value | Consumption |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Geral | 12345 | Imported Active Energy | 2024-10-01 00:00:00 | 2024 | 10 | 1 | 0 | 0 | 2738,2201 | 0,0113 |
| Geral | 12345 | Imported Active Energy | 2024-10-01 00:05:00 | 2024 | 10 | 1 | 0 | 5 | 2738,2311 | 0,011 |
| Geral | 12345 | Imported Active Energy | 2024-10-01 00:10:00 | 2024 | 10 | 1 | 0 | 10 | 2738,2422 | 0,0111 |

### 1b. Generation Structure Example

| Device | TagId | TagName | Date | A | M | D | H | m | Value | Generation |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| Geral | 54321 | Total | 2024-10-01 00:00:00 | 2024 | 10 | 1 | 0 | 0 | 11759,6907 | 0,0 |
| Geral | 54321 | Total | 2024-10-01 00:05:00 | 2024 | 10 | 1 | 0 | 5 | 11759,6907 | 0,0 |
| Geral | 54321 | Total | 2024-10-01 00:10:00 | 2024 | 10 | 1 | 0 | 10 | 11759,6907 | 0,0 |

### 2. Column Definitions

**[Mandatory]** **Time & Data:** `A` (Year), `M` (Month), `D` (Day), `H` (Hour), `m` (Minute), and `Consumption`/`Generation`.

**[Optional]** **Metadata:** `Device`, `TagId`, `TagName`, `Date`, `Value` (Useful for auditing but not processed).

> **Forecasting Note:** While the system accepts various data structures, the forecasting service is currently optimized to provide accurate forecasts only for data representing **Imported Active Energy** or **Total (PV generation)** with 5 minute granularity.

## Endpoints

### **POST** `/consumptionForecast`

Triggers the full consumption forecast workflow (data processing, train, test, and predict).

### **POST** `/pvGenerationForecast`

Triggers the full pv generation forecast (data processing, train, test, and predict).

### Request Body (multipart/form-data)

| Key | Type | Required | Description |
|---|---|---|---|
| `file` | File | Yes | The `.xlsx` dataset file. |
| `start_date` | Text | No | The start date for the prediction time frame. |
| `end_date` | Text | No | The end date for the prediction time frame. |

> **Time frame restrictions:** Maximum time frame is 14 days and minimum timeframe is 1 hour.

### Responses

#### **200 OK**

Returns a JSON object containing the prediction data as a dictionary.

```json
{
    "status": "success",
    "message": "Pipeline executed successfully",
    "forecast_data": predictions_data_dict,
    "algorithm_metrics": metrics_dict
}

```

## User Guide: Postman

Follow these steps to test the API using Postman.

1. **Create Request**: Set method to `POST` and URL to
   `http://gecad.isep.ipp.pt/api/mast/clw_forecast_api/consumptionForecast`.

**OR**

2. **Create Request**: Set method to `POST` and URL to
   `http://gecad.isep.ipp.pt/api/mast/clw_forecast_api/pvGenerationForecast`.

3. **Body Configuration**:
   * Go to the **Body** tab.
   * Select **form-data**.

4. **Set Keys**:

| Key | Value | Note |
| :--- | :--- | :--- |
| `file` | [Select File] | Hover over the "Key" field and change type from "Text" to "File". |
| `start_date` | YYYY-mm-dd HH:mm:ss | The date must be in the following format: YYYY-mm-dd HH:mm:ss. Example: 2024-12-25 00:00:00 |
| `end_date` | YYYY-mm-dd HH:mm:ss | The date must be in the following format: YYYY-mm-dd HH:mm:ss. Example: 2024-01-06 12:30:30 |

5. **Send & Download**: Click the arrow next to "Send" and choose **"Send and Download"** to save the Excel file.

---

## User Guide: cURL

You can execute the pipeline directly from the command line.

### Trigger Pipeline

```bash
# Example: Triggering a forecast for a specific 24-hour window
curl -X POST "http://gecad.isep.ipp.pt/api/mast/clw_forecast_api/consumptionForecast" \
  -H "accept: application/json" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@/path/to/your/dataset.xlsx" \
  -F "start_date=2025-01-01 00:00:00" \
  -F "end_date=2025-01-02 00:00:00"
```

## GECAD/CLW Energy Optimization Service

This API provides a Mixed-Integer Linear Programming (MILP) solver interface to optimize energy management strategies for prosumers, including battery storage and grid interaction.

## Optimize Energy Flow

**POST** `/optimize`

Uploads an Excel configuration file, runs the optimization model, and returns a results file containing optimized trajectories for battery and grid usage.

### Request Body

The request must be sent as `multipart/form-data`.

| Key | Type | Description |
| :--- | :--- | :--- |
| `file` | File (.xlsx) | The Excel file containing load, PV, and constraint data. |

## Input Excel Requirements

The input file must be a `.xlsx` file with the following sheet names:

| Sheet Name | Description |
| :--- | :--- |
| `Load` | Power consumption profile (kW). |
| `PV` | Photovoltaic generation profile (kW). |
| `Limits` | Grid import/export constraints and fixed costs. |
| `Bat` | Battery technical specs (Capacity, Charging rates). |
| `Buy_price` | Time-of-Use (ToU) grid prices. |
| `Sell_price` | Feed-in-Tariff (FiT) prices. |

## Output Excel Format

The service returns a multi-sheet Excel file (`optimized_results.xlsx`) containing:

* **Load / PV:** Copies of the input data for reference.
* **Import / Export:** Optimized grid interaction schedules.
* **Bat_ch / Bat_dch:** Battery charge and discharge schedules.
* **Est_bat:** State of Charge (SoC) trajectory.
* **Energy_bill:** Financial breakdown per period.
* **Summary:** Objective value (total cost) and execution time.

## Example Request (cURL)

```bash
curl -X 'POST' \
  'http://localhost:8000/optimize' \
  -H 'accept: application/json' \
  -H 'Content-Type: multipart/form-data' \
  -F 'file=@your_input_file.xlsx'
```

> **Tip:** If using **Postman**, ensure the Key is set to `file` and the type is set to `File` in the Body > form-data tab.

## Error Codes

| Status | Meaning | Solution |
| :--- | :--- | :--- |
| `400` | Bad Request | Incorrect file format (must be .xlsx). |
| `422` | Unprocessable Entity | Form-data key is missing or not named 'file'. |
| `500` | Internal Server Error | Check Excel sheet names or solver constraints. |
