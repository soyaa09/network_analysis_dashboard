import tkinter as tk
from tkinter import ttk, messagebox
import pandas as pd
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import numpy as np

plt.style.use("dark_background")

# =========================
# FILE CONFIG
# =========================
FILES = {
    "Gujarat": {
        "4G": "Gujarat_4G.xlsx",
        "5G": "Gujarat_5G.xlsx"
    },
    "Maharashtra": {
        "4G": "Maharashtra_4G.xlsx",
        "5G": "Maharashtra_5G.xlsx"
    }
}

METRICS = {
    "Download Speed (Mbps)": ("avg_d_mbps", "Mbps"),
    "Upload Speed (Mbps)": ("avg_u_mbps", "Mbps"),
    "Latency (ms)": ("avg_lat_ms", "ms")
}

# =========================
# MAIN WINDOW
# =========================
root = tk.Tk()
root.title("📡 Network Performance Analysis")
root.geometry("750x750")
root.configure(bg="#020617")

style = ttk.Style()
style.theme_use("clam")
style.configure(".", background="#020617", foreground="white", fieldbackground="#020617")
style.configure("TLabel", font=("Segoe UI", 10))
style.configure("Header.TLabel", font=("Segoe UI", 22, "bold"), foreground="#38bdf8")
style.configure("TLabelframe", background="#020617", foreground="#e5e7eb")
style.configure("TLabelframe.Label", background="#020617", foreground="#e5e7eb")

# =========================
# TITLE
# =========================
ttk.Label(
    root,
    text="📡 Network Performance Analysis Dashboard",
    style="Header.TLabel"
).pack(pady=12)

# =========================
# LAYOUT
# =========================
main_frame = ttk.Frame(root)
main_frame.pack(fill=tk.BOTH, expand=True, padx=16, pady=10)

control = ttk.Frame(main_frame, width=320)
control.pack(side=tk.LEFT, fill=tk.Y, padx=(0, 14))

graph_frame = tk.Frame(main_frame, bg="#020617")
graph_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)

# =========================
# REGION
# =========================
region_frame = ttk.Labelframe(control, text="🌍 Region", padding=8)
region_frame.pack(fill=tk.X, pady=6)

region_var = tk.StringVar(value="Both Regions")
ttk.Combobox(
    region_frame,
    textvariable=region_var,
    values=["Gujarat", "Maharashtra", "Both Regions"],
    state="readonly"
).pack(fill=tk.X)

# =========================
# NETWORK TYPE
# =========================
net_frame = ttk.Labelframe(control, text="📶 Network Type", padding=8)
net_frame.pack(fill=tk.X, pady=6)

net_4g = tk.BooleanVar(value=True)
net_5g = tk.BooleanVar(value=True)

ttk.Checkbutton(net_frame, text="4G", variable=net_4g).pack(anchor="w")
ttk.Checkbutton(net_frame, text="5G", variable=net_5g).pack(anchor="w")

# =========================
# GRAPH TYPE
# =========================
graph_type_frame = ttk.Labelframe(control, text="📈 Graph Type", padding=8)
graph_type_frame.pack(fill=tk.X, pady=6)

graph_var = tk.StringVar(value="Bar Chart")
ttk.Combobox(
    graph_type_frame,
    textvariable=graph_var,
    state="readonly",
    values=[
        "Bar Chart",
        "Line Graph",
        "Boxplot",
        "Congestion Scatter",
        "Heatmap"
    ]
).pack(fill=tk.X)

# =========================
# METRICS
# =========================
metric_frame = ttk.Labelframe(control, text="📊 Metrics", padding=5)
metric_frame.pack(fill=tk.BOTH, pady=2python, expand=True)

metric_list = tk.Listbox(
    metric_frame,
    selectmode=tk.MULTIPLE,
    height=2,
    exportselection=False,
    bg="#020617",
    fg="white"
)
for m in METRICS.keys():
    metric_list.insert(tk.END, m)
metric_list.pack(fill=tk.BOTH, expand=True)

# =========================
# PLOT FUNCTION
# =========================
def plot_graph():
    for w in graph_frame.winfo_children():
        w.destroy()

    if not net_4g.get() and not net_5g.get():
        messagebox.showerror("Error", "Select at least one network type!")
        return

    selected = metric_list.curselection()
    if not selected:
        messagebox.showerror("Error", "Select at least one metric!")
        return

    selected_metrics = [metric_list.get(i) for i in selected]

    limit = 150  # ✅ FIXED STANDARD SAMPLE SIZE

    regions = ["Gujarat", "Maharashtra"] if region_var.get() == "Both Regions" else [region_var.get()]

    fig, ax = plt.subplots(figsize=(9.5, 6))
    fig.patch.set_facecolor("#020617")
    ax.set_facecolor("#020617")

    colors = {
        ("Gujarat", "4G"): "#38bdf8",
        ("Gujarat", "5G"): "#f97316",
        ("Maharashtra", "4G"): "#22c55e",
        ("Maharashtra", "5G"): "#fb7185"
    }

    # ================= BAR CHART =================
    if graph_var.get() == "Bar Chart":
        labels, values, bar_colors = [], [], []

        for r in regions:
            for m in selected_metrics:
                col, unit = METRICS[m]
                if net_4g.get():
                    df = pd.read_excel(FILES[r]["4G"]).head(limit)
                    labels.append(f"{r} 4G - {m}")
                    values.append(df[col].mean())
                    bar_colors.append(colors[(r, "4G")])
                if net_5g.get():
                    df = pd.read_excel(FILES[r]["5G"]).head(limit)
                    labels.append(f"{r} 5G - {m}")
                    values.append(df[col].mean())
                    bar_colors.append(colors[(r, "5G")])

        bars = ax.bar(labels, values, color=bar_colors)
        ax.bar_label(bars, fmt="%.2f", padding=3)
        ax.set_title("Average Metrics Comparison")
        ax.set_xticklabels(labels, rotation=25, ha="right")

    # ================= LINE GRAPH =================
    elif graph_var.get() == "Line Graph":
        for r in regions:
            for m in selected_metrics:
                col, unit = METRICS[m]
                if net_4g.get():
                    df = pd.read_excel(FILES[r]["4G"]).head(limit)
                    ax.plot(df.index, df[col], label=f"{r} 4G - {m}",
                            color=colors[(r, "4G")])
                if net_5g.get():
                    df = pd.read_excel(FILES[r]["5G"]).head(limit)
                    ax.plot(df.index, df[col], linestyle="--",
                            label=f"{r} 5G - {m}",
                            color=colors[(r, "5G")])
        ax.legend(frameon=False)

    # ================= BOXPLOT =================
    elif graph_var.get() == "Boxplot":
        data, labels = [], []
        for r in regions:
            for m in selected_metrics:
                col, unit = METRICS[m]
                if net_4g.get():
                    data.append(pd.read_excel(FILES[r]["4G"])[col].head(limit))
                    labels.append(f"{r} 4G - {m}")
                if net_5g.get():
                    data.append(pd.read_excel(FILES[r]["5G"])[col].head(limit))
                    labels.append(f"{r} 5G - {m}")

        bp = ax.boxplot(data, labels=labels, showmeans=True, patch_artist=True)
        palette = list(colors.values()) * 4
        for patch, c in zip(bp["boxes"], palette):
            patch.set_facecolor(c)
        ax.set_xticklabels(labels, rotation=25, ha="right")
        ax.set_title("Metric Distribution (Mean Shown)")

    # ================= CONGESTION SCATTER =================
    elif graph_var.get() == "Congestion Scatter":
        for r in regions:
            for m in selected_metrics:
                col, unit = METRICS[m]
                if net_4g.get():
                    df = pd.read_excel(FILES[r]["4G"]).head(limit)
                    ax.scatter(df["devices"], df[col],
                               label=f"{r} 4G - {m}",
                               color=colors[(r, "4G")], alpha=0.7)
                if net_5g.get():
                    df = pd.read_excel(FILES[r]["5G"]).head(limit)
                    ax.scatter(df["devices"], df[col],
                               marker="x",
                               label=f"{r} 5G - {m}",
                               color=colors[(r, "5G")], alpha=0.7)
        ax.legend(frameon=False)

    # ================= HEATMAP =================
    elif graph_var.get() == "Heatmap":

        if len(regions) > 1:
            messagebox.showerror("Error", "Select only one region for heatmap")
            return

        r = regions[0]
        df_list = []
        if net_4g.get():
            df_list.append(pd.read_excel(FILES[r]["4G"]).head(limit))
        if net_5g.get():
            df_list.append(pd.read_excel(FILES[r]["5G"]).head(limit))

        df_all = pd.concat(df_list)
        metric_cols = ["avg_d_mbps", "avg_u_mbps", "avg_lat_ms"]
        corr = df_all[metric_cols].corr()

        im = ax.imshow(corr, cmap="magma", vmin=-1, vmax=1)

        ax.set_xticks(range(len(metric_cols)))
        ax.set_yticks(range(len(metric_cols)))
        ax.set_xticklabels(metric_cols)
        ax.set_yticklabels(metric_cols)

        for i in range(len(metric_cols)):
            for j in range(len(metric_cols)):
                ax.text(j, i, f"{corr.iloc[i, j]:.2f}",
                        ha="center", va="center", color="black")

        fig.colorbar(im, ax=ax)
        ax.set_title(f"Correlation Heatmap – {r}")

    canvas = FigureCanvasTkAgg(fig, graph_frame)
    canvas.draw()
    canvas.get_tk_widget().pack(fill=tk.BOTH, expand=True)

# =========================
# BUTTON
# =========================
ttk.Button(control, text="🚀 Generate Graph", command=plot_graph).pack(fill=tk.X, pady=12)

root.mainloop()
