# Theoretical Foundation: AEM Water Electrolysis 3D Model

## 1. Introduction

Anion Exchange Membrane (AEM) water electrolysis is an electrochemical process for hydrogen production using alkaline electrolyte and a solid polymer membrane to separate the anode and cathode compartments. This document details the governing equations and theoretical framework implemented in the 3D multiphysics model.

## 2. Physical Domains

The AEM electrolyzer comprises four main domains:

1. **Anode (OER - Oxygen Evolution Reaction)**
   - Location: x ∈ [0, L_a]
   - Reaction: 4OH⁻ → O₂ + 2H₂O + 4e⁻  
   
2. **Cathode (HER - Hydrogen Evolution Reaction)**
   - Location: x ∈ [L_a + L_m, L_a + L_m + L_c]
   - Reaction: 2H₂O + 2e⁻ → H₂ + 2OH⁻

3. **AEM Membrane**
   - Location: x ∈ [L_a, L_a + L_m]
   - Conducts OH⁻ ions
   - Blocks water transport (selective permeability)

4. **Electrolyte**
   - KOH solution (typical: 1-8 M)
   - Fills anode and cathode compartments

## 3. Governing Equations

### 3.1 Electric Field Distribution (Potential Equation)

**Laplace Equation:**
```
∇·(σ∇φ) = 0
```

Where:
- φ = electric potential [V]
- σ = ionic conductivity [S/m]

**Boundary Conditions:**
- At anode: φ = φ_applied
- At cathode: φ = 0 (reference)
- At membrane/electrolyte interface: φ continuous

**Current Density:**
```
i = -σ∇φ   [A/m²]
```

### 3.2 Electrochemical Reactions

**Butler-Volmer Equation at Electrodes:**
```
i = i_0 {exp[α_a·η·F/RT] - exp[-α_c·η·F/RT]}
```

Where:
- i_0 = exchange current density [A/m²]
- η = overpotential [V]
- α_a, α_c = charge transfer coefficients (typically 0.5 each)
- F = Faraday constant = 96485 C/mol
- R = gas constant = 8.314 J/(mol·K)
- T = absolute temperature [K]

**Overpotential:**
```
η = φ - E_eq - i·R_contact
```

Where:
- E_eq = equilibrium potential [V]
- R_contact = contact resistance [Ω·m²]

**For OER at anode (T = 25°C, [KOH] = 8M):**
```
E_eq,OER ≈ 0.40 V vs. SHE
```

**For HER at cathode:**
```
E_eq,HER ≈ -0.83 V vs. SHE (at [KOH] = 8M)
```

### 3.3 Fluid Flow (Navier-Stokes Equations)

**Continuity Equation:**
```
∂ρ/∂t + ∇·(ρu) = 0
```

**Momentum Equation:**
```
ρ(∂u/∂t + u·∇u) = -∇p + ∇·τ + F_e + F_b
```

Where:
- ρ = fluid density [kg/m³]
- u = velocity vector [m/s]
- p = pressure [Pa]
- τ = viscous stress tensor
- F_e = electromagnetic force [N/m³]
- F_b = buoyancy force from bubbles [N/m³]

**Viscous Stress Tensor (Newtonian fluid):**
```
τ = μ(∇u + (∇u)ᵀ) - (2/3)μ(∇·u)I
```

Where μ = dynamic viscosity [Pa·s]

**Boundary Conditions:**
- At electrodes (porous): Darcy-like slip flow
- At membrane: no-penetration condition (u·n = 0)
- At outlet: p = p_atm

### 3.4 Mass Transfer (Species Transport)

**Convection-Diffusion Equation:**
```
∂c_i/∂t + ∇·(u·c_i) = ∇·(D_i∇c_i) + R_i
```

Where:
- c_i = concentration of species i [mol/m³]
- D_i = diffusion coefficient [m²/s]
- R_i = reaction rate [mol/(m³·s)]

**Key Species:**
- c_OH⁻ = hydroxide ion concentration
- c_K⁺ = potassium ion concentration
- c_H₂O = water concentration

**Charge Balance (electroneutrality):**
```
Σ(z_i·c_i) ≈ 0
```

**Current as Source:**
```
R_e = i/(n·F)   [mol/(m³·s)]
```

Where n = number of electrons transferred

**Boundary Conditions:**
- At anode: flux = i_a/(4F) [mol/(m²·s)] for O₂ generation
- At cathode: flux = i_c/(2F) [mol/(m²·s)] for H₂ generation
- At membrane: concentration gradient discontinuous, flux continuous

### 3.5 Bubble Dynamics

**Bubble Nucleation:**
At electrode surface where current density exceeds critical threshold:
```
i_bubble = i₀·exp[α·(φ - φ_eq)]
```

**Bubble Volume Fraction:**
```
ε_bubble = (Q_bubble·t_residence) / V_cell
```

Where Q_bubble = volumetric gas generation rate [m³/s]

**Gas Generation Rate (Faraday's Law):**
```
Q_gas = (i·A)/(n·F·ρ_gas)   [m³/s]
```

**Bubble Diameter (Empirical correlation):**
```
d_bubble = d_0·(i/i_0)^m
```

Typical: d_0 = 0.5-2 mm, m = 0.3-0.5

**Bubble Rise Velocity (from Stokes law, modified):**
```
u_bubble = √(g·d_bubble/8)   [m/s]
```

Where g = 9.81 m/s²

### 3.6 Heat Transfer (Energy Equation)

**Energy Balance:**
```
ρ·C_p(∂T/∂t + u·∇T) = ∇·(k∇T) + Q_source
```

Where:
- C_p = specific heat capacity [J/(kg·K)]
- k = thermal conductivity [W/(m·K)]
- Q_source = heat generation [W/m³]

**Heat Generation Sources:**

1. **Joule Heating:**
```
Q_Joule = i·(φ_anode - φ_cathode) = i·V_cell
```

2. **Reaction Heat:**
```
Q_reaction = i·ΔH_rxn/(n·F)
```
For water electrolysis at 25°C:
- ΔH_OER ≈ 286 kJ/mol (endothermic)
- ΔH_HER ≈ 286 kJ/mol (endothermic)

3. **Activation Overpotential:**
```
Q_act = i·η_act
```

**Total Heat:**
```
Q_total = Q_Joule + Q_reaction + Q_act
```

**Boundary Conditions:**
- At walls: q = h·(T - T_ambient) [W/m²] (convective cooling)
- h ≈ 10-100 W/(m²·K) (natural/forced convection)

## 4. Membrane Transport

### 4.1 Ion Transport Through Membrane

**Nernst-Planck Equation in membrane:**
```
∇·J_i = 0   (steady-state)
```

Where:
```
J_i = -D_i∇c_i - (z_i·D_i·F/RT)·c_i·∇φ
```

First term: diffusion
Second term: migration

### 4.2 Water Transport

**Osmotic Water Flux:**
```
J_water = n_water·|J_OH⁻|/(z·F)
```

n_water = water molecules per ion (typically 2-4)

**Back-diffusion:**
```
J_diffusion = -D_water·∇c_water
```

## 5. Key Performance Metrics

### 5.1 Faraday Efficiency
```
η_Faraday = (n·F·V_H₂)/(∫i·dt) × 100%
```
Where V_H₂ = measured hydrogen volume

Ideal: 100% (all current converts to H₂)
Practical: 90-98% (accounts for parasitic reactions)

### 5.2 Energy Consumption

**Thermodynamic minimum:**
```
W_th = ΔG/(n·F) = 1.23 V (at 25°C, 1 atm)
```

**Actual voltage requirement:**
```
V_cell = E_eq + η_act,anode + η_act,cathode + η_ohmic + η_diffusion
```

**Specific energy consumption:**
```
SEC = (V_cell·I)/(n_H₂)   [kWh/kg H₂]
```

Practical range: 50-60 kWh/kg

### 5.3 Current Efficiency and Distribution

**Local current efficiency:**
```
η_local = i_faradaic/i_total
```

**Average current density:**
```
i_avg = I_total/A_electrode
```

## 6. Coupling Strategy

### 6.1 Two-Way Coupling
```
Electric Potential (φ)
        ↓
    → Current Density (i)
        ↓
Electrochemical Reaction Rate
        ↓
Species Generation/Consumption
        ↓
Concentration (c_i)
        ↓
Conductivity σ = σ(c_i)
        ↑ (feedback)
```

### 6.2 Multiphysics Integration

1. **Solve**: Laplace equation for φ
2. **Extract**: Current density i from potential gradient
3. **Solve**: Navier-Stokes for flow field u
4. **Solve**: Species transport with reaction source from i
5. **Solve**: Energy equation with Joule heating from i
6. **Update**: Material properties (σ, μ, k) from T and c
7. **Iterate**: Until convergence

## 7. Dimensionless Numbers

### 7.1 Peclet Number (advection vs. diffusion)
```
Pe = u·L/D
```
- Pe << 1: Diffusion dominated
- Pe >> 1: Advection dominated

### 7.2 Damköhler Number (reaction vs. convection)
```
Da = k·L²/u·L = k·L/u
```

### 7.3 Rayleigh Number (buoyancy-driven flow)
```
Ra = g·β·ΔT·L³/(ν·α)
```

## 8. Model Assumptions

1. **Isothermal electrochemistry** (except in thermal subdomain)
2. **Rigid electrode matrix** (no deformation)
3. **No side reactions** (all current goes to main reaction)
4. **Perfect mixing in bulk electrolyte**
5. **Linear polarization curves** (small overpotentials)
6. **Binary electrolyte** (KOH/H₂O)
7. **Porous electrodes** treated as continuum
8. **Bubble coalescence** neglected

## 9. Numerical Approach

### 9.1 Spatial Discretization
- **Finite Element Method (FEM)** via COMSOL
- Tetrahedral elements with local refinement
- Higher-order elements near electrodes and membrane

### 9.2 Temporal Discretization
- Implicit backward Euler (BDF) scheme
- Time-dependent problems: 0.01-0.1 s time steps
- Steady-state: Direct solver

### 9.3 Solver Strategy
- **Electric**: Direct sparse solver (UMFPACK/PARDISO)
- **Flow**: GMRES with multigrid preconditioner
- **Transport**: GMRES with ILU preconditioner
- **Coupling**: Fully coupled solver (Jacobian + Newton)

## 10. Validation Criteria

1. **Current Conservation:**
```
∮ i·dA = I_total (at any cross-section)
```

2. **Charge Balance:**
```
Σ(z_i·c_i) ≈ constant
```

3. **Energy Balance:**
```
I·V_cell = ∫(Q_Joule + Q_reaction)dV + Heat loss
```

## 11. References

- Newman, J., & Alyea, K. E. (2004). Electrochemical Systems (3rd ed.)
- Oloman, C. (2009). Electrochemistry and the Environment
- Srinivasan, S. (2006). Fuel Cells: From Fundamentals to Applications

## 12. Nomenclature

| Symbol | Description | Unit |
|--------|-------------|------|
| φ | Electric potential | V |
| σ | Ionic conductivity | S/m |
| i | Current density | A/m² |
| η | Overpotential | V |
| T | Temperature | K |
| c_i | Concentration of species i | mol/m³ |
| D_i | Diffusion coefficient | m²/s |
| u | Velocity vector | m/s |
| p | Pressure | Pa |
| ρ | Fluid density | kg/m³ |
| k | Thermal conductivity | W/(m·K) |
| C_p | Specific heat capacity | J/(kg·K) |

---

**Last Updated:** 2026-04-10
**Author:** AEM Electrolyzer Model Team