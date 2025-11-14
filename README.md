# practica-6-al071869
Modelados de problemas aplicados a la Ingeniería Civil
# ============================================================
#  DISEÑO DE ESPESOR DE CARPETA ASFÁLTICA (AASHTO 1993)
#  Versión con interfaz gráfica (Tkinter)
# ============================================================

import math
import tkinter as tk
from tkinter import ttk, messagebox

# ------------------------------------------------------
# FUNCIONES DEL MÉTODO AASHTO 1993
# ------------------------------------------------------

def aashto_logW18_from_SN(SN, Zr, S0, delta_PSI, MR):
    term1 = Zr * S0
    term2 = 9.36 * math.log10(SN + 1.0)
    term3 = -0.20
    denom = 0.4 + 1094.0 / ((SN + 1.0) ** 5.19)
    term4 = math.log10(delta_PSI / 2.7) / denom
    term5 = 2.32 * math.log10(MR)
    term6 = -8.07
    return term1 + term2 + term3 + term4 + term5 + term6

def find_SN_for_W18(W18, Zr, S0, delta_PSI, MR, tol=1e-6):
    logW18_target = math.log10(W18)
    SN_min, SN_max = 0.01, 30.0
    fa = aashto_logW18_from_SN(SN_min, Zr, S0, delta_PSI, MR) - logW18_target
    fb = aashto_logW18_from_SN(SN_max, Zr, S0, delta_PSI, MR) - logW18_target

    if fa * fb > 0:
        raise ValueError("No hay cambio de signo: revisa tus parámetros (quizá W18 es muy grande o pequeño).")

    for _ in range(300):
        SN_mid = (SN_min + SN_max) / 2
        fm = aashto_logW18_from_SN(SN_mid, Zr, S0, delta_PSI, MR) - logW18_target
        if abs(fm) < tol:
            return SN_mid
        if fa * fm < 0:
            SN_max = SN_mid
            fb = fm
        else:
            SN_min = SN_mid
            fa = fm
    return SN_mid

def SN_to_HMA_thickness_mm(SN, a1=0.44):
    D_in = SN / a1
    D_mm = D_in * 25.4
    return D_in, D_mm

# ------------------------------------------------------
# FUNCIÓN PRINCIPAL DEL BOTÓN "CALCULAR"
# ------------------------------------------------------

def calcular():
    try:
        W18 = float(entry_W18.get())
        MR = float(entry_MR.get())
        Zr = float(entry_Zr.get())
        S0 = float(entry_S0.get())
        pi = float(entry_pi.get())
        pt = float(entry_pt.get())
        a1 = float(entry_a1.get())

        delta_PSI = pi - pt

        SN = find_SN_for_W18(W18, Zr, S0, delta_PSI, MR)
        D_in, D_mm = SN_to_HMA_thickness_mm(SN, a1)

        result_SN.config(text=f"{SN:.3f}")
        result_Din.config(text=f"{D_in:.2f} in")
        result_Dmm.config(text=f"{D_mm:.1f} mm")

    except Exception as e:
        messagebox.showerror("Error de cálculo", str(e))

# ------------------------------------------------------
# INTERFAZ GRÁFICA (Tkinter)
# ------------------------------------------------------

root = tk.Tk()
root.title("Diseño de Espesor de Carpeta Asfáltica - AASHTO 1993")
root.geometry("600x500")
root.configure(bg="#f0f4f7")

title = tk.Label(root, text="Diseño de Carpeta Asfáltica (AASHTO 1993)",
                 font=("Arial", 16, "bold"), bg="#003366", fg="white", pady=10)
title.pack(fill="x")

frame = ttk.Frame(root, padding=20)
frame.pack(fill="both", expand=True)

# Campos de entrada
labels = ["Tráfico acumulado W18 (ESALs):", 
          "Módulo resiliente MR (psi):",
          "Zr (confiabilidad):",
          "S0 (error combinado):",
          "Serviciabilidad inicial (pi):",
          "Serviciabilidad final (pt):",
          "Coeficiente estructural a1:"]
entries = []

for i, text in enumerate(labels):
    ttk.Label(frame, text=text).grid(row=i, column=0, sticky="w", pady=5)
    e = ttk.Entry(frame, width=20)
    e.grid(row=i, column=1, pady=5)
    entries.append(e)

entry_W18, entry_MR, entry_Zr, entry_S0, entry_pi, entry_pt, entry_a1 = entries

# Botón calcular
ttk.Button(frame, text="Calcular", command=calcular).grid(row=7, column=0, columnspan=2, pady=15)

# Resultados
ttk.Label(frame, text="Número Estructural (SN):", font=("Arial", 10, "bold")).grid(row=8, column=0, sticky="w", pady=5)
result_SN = ttk.Label(frame, text="—", font=("Arial", 10))
result_SN.grid(row=8, column=1, sticky="w")

ttk.Label(frame, text="Espesor de carpeta:", font=("Arial", 10, "bold")).grid(row=9, column=0, sticky="w", pady=5)
result_Din = ttk.Label(frame, text="—", font=("Arial", 10))
result_Din.grid(row=9, column=1, sticky="w")

result_Dmm = ttk.Label(frame, text="—", font=("Arial", 10))
result_Dmm.grid(row=10, column=1, sticky="w")

root.mainloop()