name: RDP via Ngrok

on: [push, workflow_dispatch]

jobs:
  setup:
    runs-on: windows-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Download Ngrok
      run: Invoke-WebRequest -Uri 'https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-windows-amd64.zip' -OutFile ngrok.zip

    - name: Extract Ngrok
      run: Expand-Archive ngrok.zip

    - name: Auth Ngrok
      run: .\ngrok\ngrok.exe authtoken ${{ secrets.NGROK_AUTH_TOKEN }}

    - name: Allow RDP
      run: |
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
        Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "YourPasswordHere" -Force)

    - name: Start Ngrok Tunnel
      run: Start-Process -NoNewWindow -FilePath ".\ngrok\ngrok.exe" -ArgumentList "tcp 3389"

    - name: Wait for Connections
      run: |
        Start-Sleep -Seconds 3600
