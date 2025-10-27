# mixed-fermentation-rl

# 🧬 Mixed Fermentation Model with Reinforcement Learning

[![Open In Colab:(https://colab.research.google.com/drive/1r7pZS7nxVnHnZjRoGZcgHLNB82MOQLYr?usp=sharing))
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

## 📋 Overview

Hybrid mathematical model for **biological-chemical mixed fermentation** in industrial baking, enhanced with **Reinforcement Learning** and **Differential Evolution** for multi-objective optimization.

### Key Features

- 🔬 **Hybrid Kinetic Model**: Monod (biological) + Arrhenius (chemical)
- 🤖 **Dual Optimization**: Q-Learning + Differential Evolution
- 📊 **ML Quality Prediction**: Random Forest (R²=0.968)
- 📈 **Sensitivity Analysis**: Morris method for parameter ranking
- 💰 **Economic Analysis**: Cost-benefit with time-value considerations
- 📚 **Full Reproducibility**: Seed=42, documented parameters

---

## 🎯 Validated Results

| Method | Volume (%) | Quality | CO₂ Bio (mL) | CO₂ Chem (mL) | Efficiency | Reward |
|--------|-----------|---------|--------------|---------------|------------|--------|
| **Baseline** | 171.6 | **83.4** | 190.7 | 370.2 | 21.3 | - |
| **Q-Learning** | 193.3 | 78.0 | 190.7 | 641.6 | 27.0 | 0.677 |
| **Diff. Evolution** | **218.5** | 80.0 | 274.0 | 720.0 | **29.9** | **0.740** |

✅ All metrics validated against literature (μ_max, CO₂ yield, expansion)

---

## 🚀 Quick Start

### Installation
```bash
# Clone repository
git clone https://github.com/Lucas-Medeiros29/mixed-fermentation-rl.git
cd mixed-fermentation-rl

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Or install as package
pip install -e .
```

### Basic Usage
```python
from src.fermentation_model import MixedFermentationModel

# Initialize model
model = MixedFermentationModel()

# Define parameters
params = {
    'm_bio': 14,      # Biological yeast (g)
    'm_chem': 4.5,    # Chemical yeast (g)
    'm_sugar': 12,    # Sugar (g)
    'T_ferm': 28,     # Temperature (°C)
    't_ferm': 90,     # Time (min)
}

# Run simulation
results = model.simulate(params)

print(f"Volume: {results['V_total']:.1f}%")
print(f"Quality: {results['quality']:.1f}/100")
print(f"CO₂ Bio: {results['CO2_bio']:.1f} mL")
print(f"CO₂ Chem: {results['CO2_chem']:.1f} mL")
```

**Output:**
Volume: 171.6%
Quality: 83.4/100
CO₂ Bio: 190.7 mL
CO₂ Chem: 370.2 mL

---

## 📊 Model Architecture

### Mathematical Foundation

**Biological Fermentation (Monod)**
dX/dt = μ(S,T) · X
dS/dt = -(1/Yₓₛ) · μ(S,T) · X
dP/dt = Yₚₓ · μ(S,T) · X · η
μ(S,T) = μₘₐₓ · S/(Kₛ+S) · exp(-((T-28)/12)²)

**Chemical Fermentation (Simplified Kinetic)**
CO₂(t) = m_chem · Y_CO₂ · [0.3·(t/5) if t<5 else 0.3+0.7·(1-e^(-k·(t-5)))]

**Multi-Objective Reward (RL)**
R = 0.35·(V/V_target) + 0.30·(Q/100) + 0.25·(1-C/C_max) + 0.10·E

### Parameter Values

| Parameter | Value | Unit | Source |
|-----------|-------|------|--------|
| μ_max | 0.5 | h⁻¹ | Calibrated |
| K_s | 0.2 | g/L | Literature |
| Y_px | 8.5 | mL/g | Calibrated |
| α_bio | 0.22 | %/mL | Validated |
| α_chem | 0.08 | %/mL | Validated |

---

## 📈 Examples

### Optimization with RL
```python
from src.optimization import optimize_with_rl

# Define targets
targets = {
    'V_target': 180,  # Target volume (%)
    'C_max': 3.0,     # Max cost (R$)
    'E_target': 40    # Target efficiency
}

# Optimize
optimal_params, history = optimize_with_rl(
    initial_params=params,
    targets=targets,
    episodes=100
)

print(f"Optimal Bio: {optimal_params['m_bio']:.1f}g")
print(f"Optimal Chem: {optimal_params['m_chem']:.1f}g")
print(f"Final Reward: {optimal_params['reward']:.3f}")
```

### Sensitivity Analysis
```python
from src.sensitivity import sensitivity_analysis

df_sens = sensitivity_analysis(params, n_samples=80)

# Plot results
from src.visualization import plot_sensitivity
plot_sensitivity(df_sens)
```

### Quality Prediction (ML)
```python
from src.fermentation_model import train_quality_predictor

model_ml, scaler, cv_scores = train_quality_predictor(n_samples=600)

print(f"R² Score: {cv_scores.mean():.3f} ± {cv_scores.std():.3f}")
print(f"Feature Importances:")
for feat, imp in zip(['Bio', 'Chem', 'Sugar', 'Temp', 'Time'], 
                     model_ml.feature_importances_):
    print(f"  {feat}: {imp:.3f}")
```

---

## 📚 Documentation

- [Installation Guide](docs/installation.md)
- [Quick Start Tutorial](docs/quickstart.md)
- [API Reference](docs/api_reference.md)
- [Mathematical Model](docs/mathematical_model.md)
- [Tutorials](docs/tutorials/)

---

## 🔬 Scientific Applications

**Suitable for:**
- Industrial bakery optimization
- Product development (bread, pizza, cakes)
- Quality control and prediction
- Process automation
- Academic research

**Extensible to:**
- Sourdough (multi-strain fermentation)
- Temperature profiles (ramp, oscillatory)
- pH control systems
- Continuous fermentation
- Scale-up modeling

---

## 📊 Benchmarks

Performance on Intel i7-11700K, 16GB RAM:

| Operation | Time | Iterations |
|-----------|------|------------|
| Single simulation | ~15ms | 1 |
| Sensitivity analysis | ~1.2s | 400 |
| RL optimization | ~8s | 100 episodes |
| DE optimization | ~6s | 80 iterations |
| ML training (RF) | ~2s | 600 samples |

---

## 🧪 Testing
```bash
# Run all tests
pytest tests/ -v

# Run specific test
pytest tests/test_model.py::test_simulation -v

# With coverage
pytest tests/ --cov=src --cov-report=html
```

---

## 📄 Citation

If you use this code in your research, please cite:
```bibtex
@software{mixed_fermentation_rl_2025,
  author = {Lucas Fabrício Medeiros},
  title = {Mixed Fermentation Model with Reinforcement Learning},
  year = {2025},
  url = {https://github.com/Lucas-Medeiros29/mixed-fermentation-rl},
  version = {1.0.0}
}

---

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## 📝 License

MIT License

---

## 👥 Author

- **Lucas Fabrício Medeiros** - *Initial work* -
- GitHub: (https://github.com/LUCAS-MEDEIROS29)

---

## 🙏 Acknowledgments

- Inspired by classical fermentation engineering (Monod, 1949)
- Q-Learning algorithm (Watkins & Dayan, 1992)
- Scientific Python community (NumPy, SciPy, scikit-learn)

---

## 📞 Contact

- Email: lucasfabriciomedeiros@gmail.com
- Issues: [GitHub Issues](https://github.com/Lucas-Medeiros29/mixed-fermentation-rl/issues)

---

## 🔮 Roadmap

- [ ] v1.1: Deep RL (DQN, PPO) implementation
- [ ] v1.2: Real-time dashboard (Streamlit)
- [ ] v1.3: Multi-product optimization
- [ ] v2.0: Experimental validation module
- [ ] v2.1: IoT sensor integration

---

<div align="center">

**Made with ❤️ for the food science and AI community**

⭐ Star this repo if you find it useful!

</div>
