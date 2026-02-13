# Application Setup Guide

This guide provides step-by-step instructions to set up and run both the Backend (Java/Maven) and Frontend (Python/Streamlit) applications.

---

## Backend Setup

### 1. Create RDS Instance
Create an RDS database instance before proceeding with backend setup.



### 1. Install Dependencies
```bash
sudo yum install git -y
sudo yum install maven -y
```
### 3. Clone Repository
```bash
git clone https://github.com/CloudTechDevOps/Java-springboot-project.git
cd Java-springboot-project/backend
```
### 4. Configure RDS Details
Update the RDS connection details in:
```
/src/main/resources/application.properties
```
Ensure the database URL, username, and password match your RDS instance configuration.

### 5. Build Application
Run Maven build command from the backend directory:
```bash
mvn clean package -DskipTests=true
```

### 6. Navigate to Target Directory
```bash
cd target
```

### 7. Run JAR File
```bash
nohup java -jar datastore-0.0.7.jar > ~/datastore-nohup.out 2>&1 &
```
The backend application will now run on port **8084** (default configuration).

---

## Frontend Setup

### 1. Install Dependencies
```bash
sudo yum install git -y
sudo yum install python3-pip -y
```
### 2. Clone Repository
```bash
git clone https://github.com/CloudTechDevOps/Java-springboot-project.git
cd Java-springboot-project/frontend
```
### 3. Configure Backend API URL
Update the backend server IP in `app.py`:
```python
API_URL = os.environ.get("API_URL", "http://<BACKEND_PUBLIC_IP>:8084")
```
Replace `<BACKEND_PUBLIC_IP>` with the public IP address of your backend server. Keep the port as **8084**.

### 4. Install Python Dependencies
```bash
pip install -r requirements.txt
```

### 5. Run Application
```bash
streamlit run app.py
```
The frontend application will start on port **8501**.

---

## Access the Application

Once both services are running:

2. **Frontend Application**: `http://<FRONTEND_PUBLIC_IP>:8501`

Open your browser and navigate to the frontend URL using the public IP and port 8501.

---
