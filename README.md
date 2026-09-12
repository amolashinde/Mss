import pandas as pd


class SolarMaintenanceAnalyzer:

    def create_inspections_df(self, inspection_data: list) -> pd.DataFrame:

        columns = [
            "InspectionID",
            "SiteID",
            "Region",
            "InspectionDate",
            "OutputMWh",
            "DowntimeHours",
            "MaintenanceStatus"
        ]

        return pd.DataFrame(inspection_data, columns=columns)

    def clean_inspection_data(self, df: pd.DataFrame) -> pd.DataFrame:

        result = df[
            df.notna().all(axis=1)
            & (df["OutputMWh"] > 0)
            & (df["DowntimeHours"] >= 0)
            & (df["MaintenanceStatus"].isin(["Completed", "Scheduled"]))
        ]

        return result[
            [
                "InspectionID",
                "SiteID",
                "Region",
                "InspectionDate",
                "OutputMWh",
                "DowntimeHours",
                "MaintenanceStatus"
            ]
        ].reset_index(drop=True)

    def add_attention_flag(
        self,
        df: pd.DataFrame,
        downtime_threshold: float
    ) -> pd.DataFrame:

        result = df.copy()

        result["NeedsAttention"] = (
            result["DowntimeHours"] > downtime_threshold
        ).astype(int)

        return result

    def site_performance_summary(
        self,
        df: pd.DataFrame
    ) -> pd.DataFrame:

        result = (
            df.groupby("SiteID")
            .agg(
                InspectionCount=("SiteID", "size"),
                TotalOutputMWh=("OutputMWh", "sum"),
                AverageDowntime=("DowntimeHours", "mean")
            )
            .reset_index()
        )

        result["AverageDowntime"] = result["AverageDowntime"].round(2)

        return result.sort_values("SiteID").reset_index(drop=True)

    def low_output_sites(
        self,
        df: pd.DataFrame,
        output_threshold: float
    ) -> pd.DataFrame:

        result = (
            df.groupby("SiteID")["OutputMWh"]
            .sum()
            .reset_index(name="TotalOutputMWh")
        )

        result = result[
            result["TotalOutputMWh"] < output_threshold
        ]

        return result.sort_values("SiteID").reset_index(drop=True)

    def regional_maintenance_cost(
        self,
        df: pd.DataFrame
    ) -> pd.DataFrame:

        completed = df[
            df["MaintenanceStatus"] == "Completed"
        ].copy()

        completed["MaintenanceCost"] = (
            completed["DowntimeHours"] * 250.0
        )

        result = (
            completed.groupby("Region")["MaintenanceCost"]
            .sum()
            .reset_index()
        )

        return result.sort_values("Region").reset_index(drop=True)





        import numpy as np


def create_temperature_array(values: list) -> np.ndarray:

    return np.asarray(values, dtype=np.float64)


def validate_temperature_array(arr: np.ndarray) -> bool:

    if arr.size == 0:
        return False

    if not np.issubdtype(arr.dtype, np.number):
        return False

    return bool(np.all((arr >= -30.0) & (arr <= 10.0)))


def compute_temperature_stats(arr: np.ndarray) -> tuple:

    mean = np.mean(arr)
    standard_deviation = np.std(arr)
    maximum = np.max(arr)
    minimum = np.min(arr)

    return (
        float(round(mean, 2)),
        float(round(standard_deviation, 2)),
        float(round(maximum, 2)),
        float(round(minimum, 2))
    )


def categorize_temperatures(arr: np.ndarray) -> np.ndarray:

    result = np.empty(arr.shape, dtype=str)

    result[(arr >= -30.0) & (arr <= -18.0)] = "Frozen"

    result[(arr > -18.0) & (arr <= 5.0)] = "Chilled"

    result[(arr > 5.0) & (arr <= 10.0)] = "Warning"

    result[(arr < -30.0) | (arr > 10.0)] = "Invalid"

    return result


def longest_warning_streak(arr: np.ndarray) -> int:

    longest = 0
    current = 0

    for value in arr:

        if 5.0 < value <= 10.0:
            current += 1
            longest = max(longest, current)
        else:
            current = 0

    return longest
