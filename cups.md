# Installation and configuration of the printer

## CUPS

```bash
sudo dnf install cups cups-filters

# Run service
sudo systemctl enable --now cups

# Add yourself to the group
sudo usermod -aG lpadmin $USER
```

Open CUPS panel in the web browser:
```bash
firefox http://localhost:631
```

Go into: `Administration -> Add Printer` and select you printer. You might need to download and install additional drivers depending on the printer.
