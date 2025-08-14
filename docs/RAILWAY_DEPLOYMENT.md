# Railway Deployment Guide for Twenty CRM

This guide explains how to deploy your Twenty CRM fork to Railway.

## Understanding the Architecture

Twenty CRM consists of three main services:
1. **Server**: The main NestJS application serving the GraphQL API and frontend
2. **Worker**: Background job processor for async tasks (emails, sync, etc.)
3. **Frontend**: React application (built and served by the server)

Plus two data services:
1. **PostgreSQL**: Primary database
2. **Redis**: Cache and queue management

## Deployment Setup

### Step 1: Prepare Your Fork

1. Make sure your fork is up to date and pushed to GitHub
2. The repository already includes Railway configuration files:
   - `railway.json` - Railway build configuration
   - `railway.toml` - Service configuration
   - `.env.railway` - Environment variable template

### Step 2: Configure Railway Services

Since you already have PostgreSQL and Redis running in Railway:

1. **Add the Twenty Server Service**:
   - In Railway, add a new service from your GitHub repo
   - Select your forked Twenty repository
   - Railway will detect the configuration files automatically

2. **Configure Environment Variables**:
   Copy the required variables from `.env.railway` to your Railway service:
   
   Essential variables:
   ```
   NODE_ENV=production
   APP_SECRET=<generate-a-secure-32-char-string>
   SERVER_URL=https://<your-railway-app-url>
   FRONTEND_URL=https://<your-railway-app-url>
   ```

   Database variables (Railway should auto-inject these):
   ```
   PG_DATABASE_URL=${{Postgres.DATABASE_URL}}
   REDIS_URL=${{Redis.REDIS_URL}}
   ```

3. **Add the Worker Service**:
   - Add another service from the same GitHub repo
   - Override the start command to: `node dist/src/queue-worker/queue-worker`
   - Set these additional environment variables:
   ```
   DISABLE_DB_MIGRATIONS=true
   DISABLE_CRON_JOBS_REGISTRATION=true
   ```
   - Copy all other environment variables from the server service

### Step 3: Build and Deploy

1. **Initial Deployment**:
   - Push your changes to GitHub
   - Railway will automatically build and deploy both services
   - The Dockerfile will:
     - Build the frontend
     - Build the backend
     - Set up the runtime environment

2. **Database Initialization**:
   - On first deployment, the server will automatically:
     - Create the database schema
     - Run migrations
     - Set up initial data

### Step 4: Verify Deployment

1. Check the deployment logs in Railway
2. Visit your app URL to see the Twenty CRM interface
3. The health check endpoint is available at: `https://your-app.railway.app/healthz`

## Common Issues and Solutions

### Port Configuration
Railway automatically sets the PORT environment variable. The app is configured to use it.

### Storage
By default, files are stored locally. For production, consider configuring S3:
```
STORAGE_TYPE=s3
STORAGE_S3_REGION=us-east-1
STORAGE_S3_NAME=your-bucket
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
```

### Email Configuration
To enable email features, configure SMTP:
```
EMAIL_DRIVER=smtp
EMAIL_SMTP_HOST=smtp.gmail.com
EMAIL_SMTP_PORT=587
EMAIL_SMTP_USER=your-email
EMAIL_SMTP_PASSWORD=your-app-password
```

### Performance Tuning
- The worker service handles background jobs
- You can scale each service independently in Railway
- Monitor memory usage, especially during builds

## Development Workflow

1. **Local Development**: 
   ```bash
   yarn start  # Runs frontend + backend + worker
   ```

2. **Push Changes**:
   ```bash
   git add .
   git commit -m "Your changes"
   git push origin main
   ```

3. **Automatic Deployment**:
   - Railway will automatically deploy on push
   - Monitor the deployment in Railway dashboard

## Monitoring

- Server logs: Check the Railway logs for the server service
- Worker logs: Check the Railway logs for the worker service
- Database: Use Railway's database tools or connect with a PostgreSQL client
- Redis: Use Railway's Redis tools or connect with a Redis client

## Next Steps

1. Configure custom domain in Railway
2. Set up monitoring (Sentry, etc.)
3. Configure email providers
4. Set up authentication providers (Google, Microsoft)
5. Enable analytics if needed