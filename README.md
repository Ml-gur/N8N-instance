# N8N-instance

After much struggle connecting WhatsApp Meta to n8n(locally), I decided to take it to the cloud instance; however, I needed to connect with Ollama Mistral, which was locally hosted. Spent 4 hours struggling, but later found a solution to connect Ollama to Cloud N8N. 
I have created a guide on some methods I found to be effective. The first step was a bit hopeful but kept failing every 3 minutestes. I hope this guide helps one of you.

The guide provides a step-by-step guide on how to connect a locally hosted Ollama instance (running in Docker) to a cloud-based n8n instance using Cloudflare Tunnel. 
It covers the necessary prerequisites, configuration steps, testing procedures, and troubleshooting tips to establish a secure connection and enable the use of local Ollama models within n8n workflows. Prerequisites 
 Docker is installed and running 
Cloudflare Tunnel (cloudflared) installed 
Cloud n8n instance access

Step 1: Check Your Existing Ollama Container 
First, check if you already have an Ollama container running: 
>docker ps 
>docker start ollama 
If you see a container conflict error when trying to create a new one, you already have an Ollama container. Choose one of these options: 
Option A: Use the Existing Container (Recommended) docker start ollama docker inspect ollama
>docker start ollama
> docker inspect ollama
Option B: Remove and Recreate (If needed) - this is the one that worked for me several times
 >docker stop ollama 
>docker rm ollama
> docker run -d ` --name ollama ` -p 11434:11434 ` -e OLLAMA_HOST=0.0.0.0 ` -v ollama:/root/.ollama ` ollama/ollama
Step 2: Verify Ollama is Running
Check that Ollama is accessible locally: 
>docker ps | findstr ollama
> Invoke-WebRequest -Uri "http://localhost:11434/api/tags" -Method GET
 If the API test fails, check the container logs: ```powershell docker logs ollama
>docker logs ollama
Step 3: Create Cloudflare Tunnel Once Ollama is confirmed working locally, create the tunnel: 
>cloudflared tunnel --url http://localhost:11434 
You'll see output like: 
+--------------------------------------------------------------------------------------------+ | Your quick Tunnel has been created! Visit it at (it may take some time to be reachable): | | https://your-unique-url.trycloudflare.com | 
+--------------------------------------------------------------------------------------------+ **Important:** Copy the tunnel URL - you'll need it for n8n configuration.

Step 4: Test Your Tunnel 
In a new PowerShell window (keep the tunnel running), test the public URL: 
>Invoke-WebRequest -Uri "https://your-unique-url.trycloudflare.com/api/tags" -Method GET 

Step 5: Configure n8n Credentials 
1. Go to your n8n cloud instance 
2. Navigate to Settings → Credentials
 3. Click "Add Credential." 
4. Select "Ollama" 
5. Configure: • Base URL: https://your-unique-url.trycloudflare.com 
• Leave all other fields empty (no authentication needed)
 6. Save the credentials

Step 6: Install Models (Optional)
 Install some models for testing: 
>docker exec ollama ollama list 
>docker exec ollama ollama pull llama3.2:1b 
>docker exec ollama ollama pull llama3.2:3b

Step 7: Test in n8n
1. Create a new workflow 
2. Add these nodes: 
• Manual Trigger 
• Ollama Chat Model
 3. Configure the Ollama Chat Model node: 
• Credentials: Select your Ollama credential 
• Model: Enter model name (e.g., llama3.2:1b)
 • Prompt: Add a test message 
4. Execute the workflow to test

Quick Status Check Script
 Use this PowerShell script to verify everything is working:
 >Write-Host "Checking Ollama container status..." 
>docker ps --filter "name=ollama"
> Write-Host "`nTesting local Ollama connection..." try { $response = Invoke-WebRequest -Uri "http://localhost:11434/api/tags" -Method GET -TimeoutSec 5 Write-Host "✓ Ollama is responding locally" -ForegroundColor Green } catch { Write-Host " Ollama is not responding locally" -ForegroundColor Red Write-Host "Error: $($_.Exception.Message)" } 
>Write-Host "`nRecent Ollama logs:" 
>docker logs --tail 10 ollama

Important Notes 
⚠️ Keep the tunnel running - Don't close the PowerShell window with cloudflared running, or your tunnel will stop.
 ⚠️ URL changes on restart - If you restart cloudflared, you'll get a new URL and need to update your n8n credentials. 
⚠️ Free tunnel limitations - Account-less tunnels have no uptime guarantee and are for testing only
