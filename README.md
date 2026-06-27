AERO-G: AI-Powered AQI Monitoring & Prediction System
AERO-G is an end-to-end Air Quality Index (AQI) monitoring, forecasting, and atmospheric stability simulation system. It focuses on analyzing and predicting air quality metrics for the industrial cities of Raipur and Bhilai in Chhattisgarh, India.

The system features:

FastAPI Backend: Serving live AQI telemetry and calculating dispersion stability.
Scikit-Learn ML Model: Random Forest model predicting AQI 24 hours into the future.
Streamlit Frontend: A dark-mode, glassmorphic UI displaying real-time parameters, historical telemetry, and AI forecasting with proactive safety alerts.
Google Cloud Run Deployment: Dockerized container ready to run as a serverless instance.
🌌 System Architecture
Mermaid diagram
🧠 Scientific Formulation: Atmospheric Stability Index (ASI)
To evaluate whether pollutants are trapped near ground level (High Gravity) or dispersing (Anti-Gravity), we compute the normalized Atmospheric Stability Index (ASI) based on the meteorological Ventilation Index (VI):

V
I
=
Wind Speed (m/s)
×
Planetary Boundary Layer (PBL) Height (m)
VI=Wind Speed (m/s)×Planetary Boundary Layer (PBL) Height (m)
Wind Speed determines the horizontal movement of air currents.
PBL Height (mixing height) determines the vertical column volume available for pollutant dispersion.
The ASI is a normalized index (
0
−
100
%
0−100%) indicating atmospheric stagnation:

A
S
I
=
100
×
e
−
V
I
V
0
ASI=100×e 
− 
V 
0
​
 
VI
​
 
 
Where 
V
0
=
1500
 m
2
/
s
V 
0
​
 =1500 m 
2
 /s is the dispersion normalization constant.

Stability Classifications:
ASI > 60: High Gravity (Trapped)
Low wind speed and low PBL height. Air is stagnant, creating an atmospheric inversion that traps particulates (PM2.5, PM10) near the surface, causing hazard spikes.
ASI < 30: Anti-Gravity (Dispersing)
High wind speed and/or high mixing height. Air is turbulent, dispersing particulates rapidly and improving ground-level air quality.
30 ≤ ASI ≤ 60: Neutral (Standard)
Normal atmospheric dispersion conditions.
🔌 API Endpoints (FastAPI)
All endpoints accept requests and return JSON payloads.

Method	Endpoint	Description
GET	/	API status check and supported cities list.
GET	/api/aqi/live/{city}	Live telemetry for raipur or bhilai including AQI, PM2.5, weather, and current ASI status.
GET	/api/aqi/forecast/{city}	24-hour forecasted AQI and stability indices using the trained Random Forest model.
GET	/api/aqi/historical/{city}	Historical 7-day weather, PM concentrations, and stability indices.
🛠️ Local Setup & Running
1. Prerequisites
Ensure you have Python 3.10+ installed on your system.

2. Install Dependencies
Install all required libraries in your active Python environment:

bash

pip install -r requirements.txt
3. Train the ML Model
Run the training script to fetch 30 days of historical weather/AQI data for Raipur and Bhilai, engineer lag features, train the Random Forest Regressor, and serialize it to disk:

bash

python ml/train.py
This saves the model binary to ml/models/aqi_forecast_model.pkl.

4. Start the Backend API
Launch the FastAPI server locally:

bash

uvicorn backend.app.main:app --host 127.0.0.1 --port 8000 --reload
You can access the interactive API docs at http://127.0.0.1:8000/docs.

5. Start the Streamlit Dashboard
Open a separate terminal window and launch the user interface:

bash

streamlit run frontend/app.py
The dashboard will open automatically in your browser at http://localhost:8501.

🐳 Dockerization & Cloud Run Deployment
The project is configured for single-container multi-process deployment, ideal for Google Cloud Run.

1. Build Docker Image
Run the docker build command (this automatically trains the model during build):

bash

docker build -t gcr.io/your-project-id/aero-g:latest .
2. Run Container Locally
Test the container locally on port 8080:

bash

docker run -p 8080:8080 gcr.io/your-project-id/aero-g:latest
Access the application at http://localhost:8080.

3. Deploy to Google Cloud Run
Push the image to Google Container Registry (GCR) or Artifact Registry and deploy:

bash

docker push gcr.io/your-project-id/aero-g:latest
gcloud run deploy aero-g \
    --image gcr.io/your-project-id/aero-g:latest \
    --platform managed \
    --region us-central1 \
    --allow-unauthenticated
Cloud Run will automatically inject the $PORT environment variable, which entrypoint.sh binds to the Streamlit frontend.
