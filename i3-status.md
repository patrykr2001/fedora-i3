# My config for i3-status

Edit `~/.config/i3status/config`:
```
# i3status configuration file.
# see "man i3status" for documentation.

general {
        colors = true
        interval = 5
}

order += "ipv6"
order += "wireless _first_"
order += "ethernet _first_"
order += "battery all"
order += "disk /"
order += "cpu_temperature 0"
order += "load"
order += "memory"
order += "tztime local"

cpu_temperature 0 {
        format = "Coffee:  %degrees°C"
        path = "/sys/class/thermal/thermal_zone0/temp"
        max_threshold = 90
        format_above_threshold = "Expresso: %degrees°C"
}

wireless _first_ {
        format_up = "W: (%quality at %essid) %ip"
        format_down = "W: down"
}

ethernet _first_ {
        format_up = "E: %ip (%speed)"
        format_down = "E: down"
}

battery all {
        format = "Unicorns: %status %percentage %remaining"
}

disk "/" {
        format = "Rule 34: %avail"
}

load {
        format = "Hamsters: %1min"
}

memory {
        format = "%used | %available"
        threshold_degraded = "1G"
        format_degraded = "%used | %available"
}

tztime local {
        format = "%Y-%m-%d %H:%M:%S"
}
```
