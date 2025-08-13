# 1. Team and Contributors  
### Team Name: ISI Foundation

### Members  
* **Davide Nicola** – Junior Researcher, ISI Foundation 

---

# 2. Repository Structure  
<pre>
your-repo-name/                            
├── data_sprint_2025/                                 # Contains surveillance and population CSV files (e.g., dengue.csv.gz, datasus_population_2001_2024.csv.gz)
├── RELATORIO_DTB_BRASIL_DISTRITOS.ods     # Municipality codes lookup for geolocation
├── main_model.py                          # Core script: data preprocessing, ODE model, Bayesian training, forecasting, submission
├── models/                                # Folder to store saved posterior results, metadata, and training data
├── forecasts/                             # Generated forecast CSVs per state
├── requirements.txt                       # Python dependencies list
└── README.md                              # This documentation file
</pre>

---

# 3. Libraries and Dependencies  
All Python dependencies are listed in `requirements.txt`. Key packages include:

- `numpy`, `pandas`, `scipy`, `matplotlib`, `arviz`  
- `pymc` (for Bayesian modeling)  
- `numba` (for accelerating ODE computations)  
- `meteostat`, `geopy`, `requests` (for weather and geolocation APIs)  
- `corner`, `pickle`, `json`, `odfpy` (supporting utilities)  

---

# 4. Data and Variables  

### Datasets Used  
We used these datasets provided by the Infodengue-Mosqlimate sprint organisers:
- **dengue.csv.gz** – Weekly dengue cases by municipality  
- **datasus_population_2001_2024.csv.gz** – Population data for normalization  
- **RELATORIO_DTB_BRASIL_DISTRITOS.ods** – Municipality names and geo‑codes for data mapping

In addition we used for weather dependet data:  
- **Weather data** – Fetched dynamically using the [Meteostat API](https://dev.meteostat.net/)

### Preprocessing Steps  
- Aggregated weekly dengue case counts from municipalityand matched with yearly population by state  
- Selected top 10 most populous cities per state for weather sampling  
- Computed rolling mean temperature, moisture balance, and derived biological parameters:
  - `theta`: Eggs development rate
  - `egg`: Eggs laying rate,
  - `bite`: Daily bite rate
- Time alignment of epidemiological and weather series ensured consistent daily-weekly mapping  

---

# 5. Model Training  

### Model Architecture  
- A vector–host SEIR ODE system for humans and mosquitoes  
- Uses temperature-dependent parameters  
- Implemented with Runge–Kutta (RK4) integration accelerated using Numba  

### Training Procedure  
- Bayesian inference via PyMC with `DEMetropolisZ` sampler  
- Priors: Beta distributions for vector and host infection probabilities  
- Likelihood: Poisson (case counts vs model predictions)  
- Posterior summaries (medians, intervals) are extracted and saved  

---

# 6. References  

- PyMC documentation: https://www.pymc.io/  
- Meteostat API docs: https://dev.meteostat.net/  

---

# 7. Data Usage Restriction  

- **Training** uses only data up to **EW 25** of each year, starting from **EW 41** of the previous year
- **Forecasting** is done from **EW 41** of the current year through **EW 40** of the following year
- This split is enforced in:
  - `fit_function()`
  - `forcast_function()`

---

# VIII. Predictive Uncertainty  

- Posterior sampling via PyMC is used to propagate uncertainty  
- We simulate weekly incidence for each posterior sample  
- Credible intervals (50%, 80%, 90%, 95%) are computed using NumPy percentiles  
- These bounds are returned in forecast CSVs inside the `forecasts/` directory  

---

