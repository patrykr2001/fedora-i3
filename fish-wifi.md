# Easy fish shell i3 WiFi CLI

Create and edit fish function file:
```bash
nano ~/.config/fish/functions/wifi.fish
```

Write:
```bash
function wifi
    if test (count $argv) -eq 0
        # Scan and list WiFi networks
        echo "Scanning WiFi networks..."
        nmcli device wifi rescan 2>/dev/null
        sleep 2
        nmcli device wifi list
        echo ""
        echo "Usage:"
        echo "  wifi connect <SSID>          # connect (will ask for password)"
        echo "  wifi connect <SSID> <hasło>  # connect with password"
        echo "  wifi status                  # current connection"
        echo "  wifi off                     # disconnect"
        return
    end

    switch $argv[1]
        case connect
            if test (count $argv) -lt 2
                echo "Enter SSID: wifi connect <SSID>"
                return 1
            end
            set ssid $argv[2]
            if test (count $argv) -ge 3
                set haslo $argv[3]
                nmcli device wifi connect $ssid password $haslo
            else
                # Check if network is already saved
                if nmcli connection show | grep -q $ssid
                    nmcli connection up $ssid
                else
                    # Ask for password
                    read -s -P "Password for $ssid: " haslo
                    echo ""
                    nmcli device wifi connect $ssid password $haslo
                end
            end

        case status
            nmcli device status | grep wifi
            echo ""
            nmcli connection show --active | grep wifi

        case off
            nmcli device disconnect (nmcli device status | grep wifi | awk '{print $1}' | head -1)
            echo "Disconnected WiFi"

        case scan
            echo "Scanning..."
            nmcli device wifi rescan
            sleep 2
            nmcli device wifi list

        case '*'
            echo "Unknown option: $argv[1]"
            echo "Usage: wifi [connect <SSID> [password] | status | off | scan]"
    end
end
```

Save and load:
```bash
source ~./config/fish/functions/wifi.fish
```

Usage:
```bash
wifi                                # Lists networks
wifi connect "NetworkName"          # Connects to network, asks for password
wifi connect "Network" "password"   # Connects to network with password
wifi status                         # Current connection
wifi off                            # Disconnect
wifi scan                           # Refresh list
```
