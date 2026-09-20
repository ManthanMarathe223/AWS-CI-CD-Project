# Student Exam Performance Indicator — Dockerized AWS CI/CD

A complete end-to-end Machine Learning application that predicts a student's **Math Score** from academic and demographic inputs. The project combines a Flask web application, a scikit-learn preprocessing/model pipeline, Docker, Amazon ECR, Amazon EC2, and GitHub Actions for CI/CD.

## Architecture

The deployment flow implemented by the project is:

<img width="1584" height="840" alt="AWS CICD Docker Deployment " src="https://github.com/user-attachments/assets/ebd79aca-d6c9-499b-8a17-b010dd02c28a" />


### Deployment flow

1. **GitHub Repository** — code is pushed to the `main` branch.
2. **GitHub Actions** — the workflow checks out the repository and builds the Docker image.
3. **Docker Image** — the application and its dependencies are packaged into a container.
4. **Amazon ECR** — the Docker image is pushed to the ECR repository.
5. **Amazon EC2** — the EC2 machine pulls the latest image from ECR.
6. **Flask ML App** — the container runs the prediction application on port `8080`.

The provided CI/CD workflow uses a self-hosted runner for the deployment stage, allowing the EC2 machine to pull the image and run the container.

---

## Project Overview

The application is an ML regression project designed to predict a student's **math score** using:

- Gender
- Race/Ethnicity
- Parental Level of Education
- Lunch Type
- Test Preparation Course
- Reading Score
- Writing Score

The Flask application accepts these values through a web form, converts them into a pandas DataFrame, loads the saved preprocessing pipeline and trained model, performs the prediction, and displays the predicted math score.

---

## Machine Learning Pipeline

### 1. Data Ingestion

The data ingestion component reads the dataset from:

```text
notebook/data/stud.csv
```

It then:

- Saves the raw dataset to `artifacts/data.csv`
- Splits the dataset into training and testing sets using an 80/20 split
- Saves the resulting datasets to:
  - `artifacts/train.csv`
  - `artifacts/test.csv`

The split uses `random_state=42`.

### 2. Data Transformation

The transformation pipeline separates features into:

**Numerical features**
- `writing_score`
- `reading_score`

**Categorical features**
- `gender`
- `race_ethnicity`
- `parental_level_of_education`
- `lunch`
- `test_preparation_course`

Numerical data is processed using median imputation and `StandardScaler`.

Categorical data is processed using most-frequent imputation, `OneHotEncoder`, and scaling.

The complete preprocessing object is saved as:

```text
artifacts/preprocessor.pkl
```

### 3. Model Training

The model trainer evaluates several regression algorithms:

- Random Forest Regressor
- Decision Tree Regressor
- Gradient Boosting Regressor
- Linear Regression
- XGBoost Regressor
- CatBoost Regressor
- AdaBoost Regressor

`GridSearchCV` is used for parameter search, and the models are evaluated using the **R² score**.

The selected trained model is saved as:

```text
artifacts/model.pkl
```

### 4. Prediction

During prediction:

1. Form data is collected by Flask.
2. `CustomData` converts the values into a DataFrame.
3. `PredictPipeline` loads:
   - `artifacts/model.pkl`
   - `artifacts/preprocessor.pkl`
4. The input is transformed using the saved preprocessor.
5. The trained model predicts the Math Score.
6. The result is displayed on the prediction page.

---

## Project Structure

```text
AWS-CI-CD-Project/
│
├── .dockerignore
├── .ebextensions/
│   └── python.config
│
├── .github/
│   └── workflows/
│       └── main.yml
│
├── artifacts/
│   ├── data.csv
│   ├── model.pkl
│   ├── preprocessor.pkl
│   ├── test.csv
│   └── train.csv
│
├── catboost_info/
│   ├── catboost_training.json
│   ├── learn_error.tsv
│   ├── learn/
│   │   └── events.out.tfevents
│   └── time_left.tsv
│
├── notebook/
│   ├── 1 . EDA STUDENT PERFORMANCE .ipynb
│   ├── 2. MODEL TRAINING.ipynb
│   ├── data/
│   │   └── stud.csv
│   └── catboost_info/
│       ├── catboost_training.json
│       ├── learn_error.tsv
│       ├── learn/
│       │   └── events.out.tfevents
│       └── time_left.tsv
│
├── src/
│   ├── __init__.py
│   ├── components/
│   │   ├── __init__.py
│   │   ├── data_ingestion.py
│   │   ├── data_transformation.py
│   │   └── model_trainer.py
│   │
│   ├── pipeline/
│   │   ├── __init__.py
│   │   ├── predict_pipeline.py
│   │   └── train_pipeline.py
│   │
│   ├── exception.py
│   ├── logger.py
│   └── utils.py
│
├── templates/
│   ├── home.html
│   └── index.html
│
├── Dockerfile
├── LICENSE
├── README.md
├── requirements.txt
├── setup.py
└── application.py
```

### File and folder purpose — one line each

| File / Folder | Purpose |
|---|---|
| `.dockerignore` | Prevents unnecessary files such as virtual environments, notebooks, Git metadata, and `.env` files from being copied into the Docker build context. |
| `.ebextensions/python.config` | Contains the Elastic Beanstalk WSGI configuration for the Flask application. |
| `.github/workflows/main.yml` | Defines the GitHub Actions CI/CD workflow that builds the Docker image, pushes it to ECR, and deploys it from the self-hosted runner. |
| `.gitignore` | Lists local, generated, environment, IDE, testing, and cache files that should not be committed to Git. |
| `application.py` | Main Flask entry point containing the web routes and prediction request handling. |
| `artifacts/data.csv` | Stored copy of the raw dataset produced during data ingestion. |
| `artifacts/train.csv` | Training split generated by the ingestion pipeline. |
| `artifacts/test.csv` | Testing split generated by the ingestion pipeline. |
| `artifacts/preprocessor.pkl` | Serialized preprocessing pipeline used to transform prediction inputs. |
| `artifacts/model.pkl` | Serialized trained regression model used for predictions. |
| `catboost_info/` | CatBoost training logs and supporting files generated during model experimentation/training. |
| `notebook/` | Jupyter notebooks and dataset used for EDA and model-training experimentation. |
| `src/components/data_ingestion.py` | Reads the source dataset, creates the train/test split, and passes the data to transformation and training components. |
| `src/components/data_transformation.py` | Builds and applies the numerical/categorical preprocessing pipelines and saves the preprocessor. |
| `src/components/model_trainer.py` | Trains and evaluates multiple regression models, performs grid search, selects a model, and saves it. |
| `src/pipeline/predict_pipeline.py` | Loads the saved model and preprocessor and performs inference on new student data. |
| `src/pipeline/train_pipeline.py` | Training-pipeline module reserved for the model-training workflow. |
| `src/exception.py` | Defines the custom exception class and formats detailed error information. |
| `src/logger.py` | Configures timestamped application logging and log-file generation. |
| `src/utils.py` | Provides reusable helpers for saving/loading objects and evaluating multiple models. |
| `templates/index.html` | Basic landing/home page template. |
| `templates/home.html` | Student prediction form and prediction-result page. |
| `Dockerfile` | Defines the Docker image build instructions for the Flask application. |
| `requirements.txt` | Lists the Python dependencies required by the project. |
| `setup.py` | Defines the Python package metadata and reads dependencies from `requirements.txt`. |
| `LICENSE` | Contains the MIT License for the project. |
| `README.md` | Project documentation, architecture, setup, usage, and deployment instructions. |

---

## Flask Application

### Routes

#### `GET /`

Loads:

```text
templates/index.html
```

This is the basic landing page.

#### `GET /predictdata`

Loads:

```text
templates/home.html
```

This displays the prediction form.

#### `POST /predictdata`

Receives the form data, creates a `CustomData` object, sends it to `PredictPipeline`, and returns the predicted Math Score to the same template.

---

## Frontend Inputs

The prediction form contains:

| Input | Type |
|---|---|
| Gender | Select |
| Race/Ethnicity | Select |
| Parental Level of Education | Select |
| Lunch Type | Select |
| Test Preparation Course | Select |
| Reading Score | Numeric |
| Writing Score | Numeric |

The prediction result is rendered through the Jinja template variable:

```text
{{ results }}
```

---

## Docker

The application is containerized using the `Dockerfile`.

Current container setup:

```text
Base image: python:3.14-slim
Working directory: /application
Application: application.py
```

The Flask application is configured to run on:

```text
0.0.0.0:8080
```

### Build the image

```bash
docker build -t student-performance .
```

### Run locally

```bash
docker run -d -p 8080:8080 --name mltest student-performance
```

Open:

```text
http://localhost:8080
```

### Stop the container

```bash
docker stop mltest
```

### Remove the container

```bash
docker rm mltest
```

---

## AWS Deployment

The project uses:

- **Amazon ECR** — Docker image registry
- **Amazon EC2** — application host
- **GitHub Actions** — CI/CD automation
- **Docker** — containerization

### Deployment process

```text
git push
   ↓
GitHub Actions
   ↓
Docker build
   ↓
Login to Amazon ECR
   ↓
Push image to ECR
   ↓
EC2 pulls latest image
   ↓
Docker container starts
   ↓
Flask ML application available on :8080
```

### ECR image

The workflow uses the `latest` tag for the deployment image:

```text
<ecr-registry>/<repository>:latest
```

### EC2 deployment

The deployment stage runs on a **self-hosted GitHub Actions runner** and performs the equivalent of:

```bash
docker pull <ECR_IMAGE>:latest
docker run -d -p 8080:8080 --name=mltest <ECR_IMAGE>:latest
docker system prune -f
```

The workflow also passes AWS environment variables into the container.

---

## GitHub Actions CI/CD

The workflow is triggered when code is pushed to the `main` branch.

### Continuous Integration

The integration job:

- Checks out the repository.
- Runs the configured lint step.
- Runs the configured test step.

### Continuous Delivery

The ECR job:

1. Checks out the repository.
2. Configures AWS credentials.
3. Logs in to Amazon ECR.
4. Builds the Docker image.
5. Tags the image as `latest`.
6. Pushes the image to ECR.

### Continuous Deployment

The deployment job runs on a self-hosted runner:

1. Checks out the repository.
2. Configures AWS credentials.
3. Logs in to ECR.
4. Pulls the latest Docker image.
5. Runs the Docker container on port `8080`.
6. Cleans unused Docker resources.

---

## GitHub Actions Secrets

The provided workflow references these repository secrets:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_ECR_LOGIN_URI
ECR_REPOSITORY_NAME
```

### Important

Never commit the actual values of these secrets into the repository.

Only the **secret names** should appear in the workflow.

---

## Local Setup

### 1. Clone the repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd AWS-CI-CD-Project
```

### 2. Create a virtual environment

Windows PowerShell:

```powershell
python -m venv venv
.env\Scriptsctivate
```

Linux/macOS:

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Run the application

```bash
python application.py
```

The application runs on:

```text
http://localhost:8080
```

---

## Training the Model

The data ingestion component contains the end-to-end training sequence:

```text
Dataset
   ↓
Data Ingestion
   ↓
Train/Test Split
   ↓
Data Transformation
   ↓
Model Training
   ↓
Model Evaluation
   ↓
Best Model
   ↓
artifacts/model.pkl
```

The preprocessing pipeline is stored separately as:

```text
artifacts/preprocessor.pkl
```

These two serialized artifacts allow the deployed Flask application to perform predictions without retraining the model on every request.

---

## Dependencies

The project currently declares:

```text
pandas
numpy
seaborn
ipykernel
scikit-learn
dill
flask
gunicorn
```

The training component also imports:

```text
catboost
xgboost
```

for model experimentation/training.

---

## Useful Commands

### Run Flask application

```bash
python application.py
```

### Build Docker image

```bash
docker build -t student-performance .
```

### Run Docker container

```bash
docker run -d -p 8080:8080 --name mltest student-performance
```

### List containers

```bash
docker ps
```

### View logs

```bash
docker logs mltest
```

### Stop container

```bash
docker stop mltest
```

### Remove container

```bash
docker rm mltest
```

### Remove unused Docker resources

```bash
docker system prune -f
```

---

## Development Workflow

A simple development workflow for this project is:

```text
Create feature branch
      ↓
Develop / test locally
      ↓
Commit changes
      ↓
Push branch
      ↓
Create Pull Request
      ↓
Merge into main
      ↓
GitHub Actions
      ↓
Docker build
      ↓
ECR push
      ↓
EC2 deployment
```

---

## Key Technologies

- **Python**
- **Flask**
- **Pandas**
- **NumPy**
- **Scikit-learn**
- **CatBoost**
- **XGBoost**
- **Jupyter Notebook**
- **Docker**
- **Git & GitHub**
- **GitHub Actions**
- **Amazon ECR**
- **Amazon EC2**

---

## Project Highlights

- End-to-end machine learning workflow
- Separate data ingestion, transformation, training, and prediction components
- Serialized model and preprocessing artifacts
- Flask web interface for predictions
- Dockerized application
- Amazon ECR container registry
- Amazon EC2 deployment
- GitHub Actions based CI/CD pipeline
- Self-hosted deployment runner

---

## Notes

The repository also contains an Elastic Beanstalk configuration under `.ebextensions/`. The current deployment architecture shown in the project documentation uses **Amazon ECR + Amazon EC2**, while the Elastic Beanstalk configuration is a separate deployment configuration retained in the repository.

The project includes both notebooks for experimentation and modular Python components for the application/training pipeline.

---

## License

This project is licensed under the **MIT License**. See the [`LICENSE`](./LICENSE) file for details.

---

## Author

**Manthan Marathe**

A machine learning project combining model development, Flask deployment, Docker containerization, and AWS-based CI/CD.
