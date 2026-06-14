# FleetFlow Deployment Guide (Vercel + Render + MongoDB Atlas)

## Architecture
- **Frontend:** React (Vercel)
- **Backend:** Node.js/Express (Render)
- **Database:** MongoDB Atlas (Managed MongoDB)

---

## STEP 1: Set Up MongoDB Atlas

1. Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
2. Sign up or log in
3. Create a new project
4. Create a new Cluster (M0 free tier is fine for testing)
5. Add a Database User with:
   - Username: `fleetflow_user`
   - Password: (generate a strong one)
   - Permissions: Read/Write
6. Add IP to Whitelist: Click "Add IP Address" → "Allow access from anywhere" (0.0.0.0/0)
7. Copy the Connection String - it will look like:
   ```
   mongodb+srv://fleetflow_user:PASSWORD@cluster.mongodb.net/fleetflow?retryWrites=true&w=majority
   ```

---

## STEP 2: Deploy Backend to Render

### 2.1: Prepare Backend

1. Update [server/.env.production](../server/.env.production) with:
   ```
   PORT=5000
   MONGO_URI=mongodb+srv://fleetflow_user:YOUR_PASSWORD@cluster.mongodb.net/fleetflow?retryWrites=true&w=majority
   JWT_SECRET=GENERATE_STRONG_SECRET_HERE
   JWT_REFRESH_SECRET=GENERATE_STRONG_SECRET_HERE
   CLIENT_URL=https://your-vercel-url.vercel.app
   NODE_ENV=production
   ```

### 2.2: Create Render Account & Deploy

1. Go to [render.com](https://render.com)
2. Sign up with GitHub
3. Click "New +" → "Web Service"
4. Connect your GitHub repository
5. Fill in the form:
   - **Name:** `fleetflow-backend`
   - **Branch:** `main`
   - **Root Directory:** `server`
   - **Runtime:** `Node`
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
6. Add Environment Variables (Settings tab):
   - `PORT`: `5000`
   - `MONGO_URI`: Your MongoDB Atlas connection string
   - `JWT_SECRET`: Your JWT secret
   - `JWT_REFRESH_SECRET`: Your JWT refresh secret
   - `CLIENT_URL`: Your future Vercel frontend URL (can update later)
   - `NODE_ENV`: `production`
7. Click "Create Web Service"
8. Wait for deployment (5-10 minutes)
9. **Copy your Render URL** (e.g., `https://fleetflow-backend.onrender.com`)

---

## STEP 3: Deploy Frontend to Vercel

### 3.1: Prepare Frontend

1. Update [client/.env.production](../client/.env.production):
   ```
   VITE_API_URL=https://fleetflow-backend.onrender.com
   ```

2. Update [client/src/api/axios.js] to use the environment variable:
   ```javascript
   const apiClient = axios.create({
     baseURL: import.meta.env.VITE_API_URL || 'http://localhost:5000',
   });
   ```

### 3.2: Create Vercel Account & Deploy

1. Go to [vercel.com](https://vercel.com)
2. Sign up with GitHub
3. Click "Add New..." → "Project"
4. Import your GitHub repository
5. Select project settings:
   - **Framework:** Vite
   - **Root Directory:** `client`
   - **Build Command:** `npm run build`
   - **Output Directory:** `dist`
6. Add Environment Variables:
   - `VITE_API_URL`: Your Render backend URL (e.g., `https://fleetflow-backend.onrender.com`)
7. Click "Deploy"
8. Wait for deployment (2-5 minutes)
9. **Copy your Vercel URL** (e.g., `https://fleetflow-client.vercel.app`)

---

## STEP 4: Update Backend CORS

Once you have your Vercel URL, update Render environment variables:

1. Go to Render dashboard → `fleetflow-backend` → Settings
2. Update `CLIENT_URL` to your Vercel URL
3. Click "Save"
4. Render will auto-redeploy

---

## STEP 5: Seed Production Database

Once everything is deployed:

```bash
# Seed data into MongoDB Atlas (run locally)
MONGO_URI="your_atlas_uri" npm run seed
```

Or SSH into Render and run:
```bash
npm run seed
```

---

## TESTING

1. **Frontend:** https://your-vercel-url.vercel.app
2. **Backend API:** https://your-render-backend.onrender.com/api/auth/login (should return a response)
3. Test login with default credentials

---

## TROUBLESHOOTING

### Backend won't start
- Check `npm start` script in package.json points to `src/index.js`
- Verify MONGO_URI is correct
- Check logs in Render dashboard

### Frontend can't reach backend
- Verify `VITE_API_URL` is set correctly
- Check CORS is enabled (backend/src/index.js)
- Verify backend URL in frontend environment

### Socket.IO connection fails
- Update Socket.IO CORS in backend: Add your Vercel URL to `CLIENT_URL`
- Check WebSocket is enabled (usually auto on Render)

---

## PRODUCTION CHECKLIST

- ✅ MongoDB Atlas cluster created & URI copied
- ✅ Backend deployed to Render with all env vars
- ✅ Frontend deployed to Vercel with VITE_API_URL set
- ✅ Backend CORS updated to frontend URL
- ✅ Database seeded with demo data
- ✅ Test login/logout flow
- ✅ Test real-time features (Socket.IO)

---

## COST

- **MongoDB Atlas:** Free M0 tier
- **Render:** $7/month (free tier has 15-min sleep limit)
- **Vercel:** Free tier
