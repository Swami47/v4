# Deployment Guide: Render (Backend) + Vercel (Frontend)

This guide walks you through deploying the **Legal Metrology Compliance Inspector** into production with:
- **Backend**: FastAPI + EasyOCR + SQLite hosted on **Render**
- **Frontend**: React + Vite SPA hosted on **Vercel**

---

## Part 1: Deploy Backend on Render

### Step 1: Push Code to GitHub
1. Initialize git and commit your repository:
   ```bash
   git init
   git add .
   git commit -m "feat: complete statutory compliance inspector"
   git remote add origin https://github.com/<your-username>/<your-repo-name>.git
   git push -u origin main
   ```

### Step 2: Create Web Service on Render
1. Go to [dashboard.render.com](https://dashboard.render.com) and sign in.
2. Click **New +** $\rightarrow$ **Web Service**.
3. Select your GitHub repository.
4. Configure the service settings:
   - **Name**: `legal-metrology-compliance-api` (or your choice)
   - **Language**: `Python 3`
   - **Branch**: `main`
   - **Root Directory**: (Leave blank / root)
   - **Build Command**:
     ```bash
     pip install --upgrade pip && pip install -r requirements.txt
     ```
   - **Start Command**:
     ```bash
     uvicorn backend.main:app --host 0.0.0.0 --port $PORT
     ```
   - **Instance Type**: Free or Starter (512MB–1GB RAM recommended for PyTorch OCR)

### Step 3: Add Environment Variables on Render
Under the **Environment Variables** section, add:
- `PYTHON_VERSION`: `3.11.8`
- `DATABASE_URL`: `sqlite:///backend/metrology_compliance.db`

### Step 4: Deploy and Copy Backend URL
1. Click **Create Web Service**.
2. Wait for the build and deployment logs to finish.
3. Test your live health endpoint:
   ```
   https://legal-metrology-compliance-api.onrender.com/api/health
   ```
4. Copy your live Render URL (e.g. `https://legal-metrology-compliance-api.onrender.com`).

---

## Part 2: Deploy Frontend on Vercel

### Step 1: Import Project on Vercel
1. Go to [vercel.com](https://vercel.com) and sign in.
2. Click **Add New...** $\rightarrow$ **Project**.
3. Select your GitHub repository.

### Step 2: Configure Project Settings
- **Framework Preset**: `Vite`
- **Root Directory**: Click **Edit** and select `frontend` (Important!)
- **Build Command**: `npm run build`
- **Output Directory**: `dist`
- **Install Command**: `npm install`

### Step 3: Set Environment Variable on Vercel
Under **Environment Variables**, add:
- **Key**: `VITE_API_BASE`
- **Value**: `https://legal-metrology-compliance-api.onrender.com` (Your live Render backend URL from Part 1, without a trailing slash)

### Step 4: Deploy
1. Click **Deploy**.
2. Vercel will build the frontend bundle and generate your live production URL (e.g. `https://legal-metrology-inspector.vercel.app`).
3. Open your Vercel URL to start running live packaging compliance audits!
