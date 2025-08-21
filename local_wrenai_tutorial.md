# **Complete Tutorial: Installing Local WrenAI on Windows**

This tutorial will guide you through setting up WrenAI locally on Windows using Docker, that you can query any database with natural language → get accurate SQL(Text-to-SQL), charts(Text-to-Charts) & AI-generated insights in seconds.

### **Prerequisites**
- Windows 10 version 2004+ or Windows 11
- Docker Desktop
- Git

## **Step 1: Clone the WrenAI repository from Github**

Open a **Windows PowerShell** Terminal and run:
```powershell
# Clone the repository
git clone https://github.com/Canner/WrenAI.git
```

## **Step 2: Copy the .env and config.yaml**

Copy the `.env` and `config.yaml` files into docker/
```powershell
cd WrenAI

# Copy the files
copy docker\env.example docker\.env
copy docker\config.example.yaml docker\config.yaml
```

*Or just copy paste those files directly from this Github repository in the docker/ directory.

## **Step 3: Running WrenAI**

After changing all the keys, endpoints and values to your Azure OpenAI credentials you can run the container using **Docker Compose**.
```powershell
# Run the app
docker compose -f docker\docker-compose.yaml up
```

## **Next Step**

Access the UI at http://localhost:3000

---

<span style="font-size:17px">Happy testing! Your local WrenAI environment is now ready for some serious development.</span>
