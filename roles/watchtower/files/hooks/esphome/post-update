#!/bin/bash
# Watchtower post-update hook

UPDATED_CONTAINER="$WATCHTOWER_CONTAINER_NAME"

# Only trigger if it's the ESPHome container
if [ "$UPDATED_CONTAINER" == "esphome" ]; then
    echo "ESPHome was updated. Running custom commands..."

    # Directory inside the container where your YAML files are mounted
    esphome_directory="/opt/esphome/config"

    # 1️⃣ Find YAML files, skip secrets.yaml, .version, and .esphome directory
    esphome_yaml_files=()
    while IFS= read -r file; do
        esphome_yaml_files+=("$file")
    done < <(find "$esphome_directory" \
        -type d -name ".esphome" -prune -o \
        -type f -name "*.yaml" ! -name "secrets.yaml" ! -name ".version" -print)

    # 2️⃣ Extract device names
    esphome_devices=()
    for file in "${esphome_yaml_files[@]}"; do
        basename="$(basename "$file")"
        device_name="${basename%.yaml}"
        esphome_devices+=("$device_name")
    done

    # 3️⃣ Resolve .local hostnames to IP addresses
    declare -A esphome_device_ips
    for device in "${esphome_devices[@]}"; do
        ip=$(ping -c 1 "$device.local" 2>/dev/null | head -1 | awk -F'[()]' '{print $2}')
        ip=${ip:-""}
        esphome_device_ips["$device"]="$ip"
    done

    # 4️⃣ Upload configs via the esphome container
    for device in "${!esphome_device_ips[@]}"; do
        ip="${esphome_device_ips[$device]}"
        if [ -n "$ip" ]; then
            echo "Uploading $device to $ip..."
            docker exec esphome esphome compile "/config/$device.yaml"
            docker exec esphome esphome upload "/config/$device.yaml" --device "$ip"
        else
            echo "Skipping $device — IP not found"
        fi
    done

    # Path to .version file
    version_file="$esphome_directory/.version"

    # Get ESPHome version from container
    esphome_version=$(docker exec esphome esphome version 2>/dev/null)

    # Check if command succeeded
    if [ $? -eq 0 ] && [ -n "$esphome_version" ]; then
        # Write version to .version file
        echo "$esphome_version" > "$version_file"
        echo "Updated .version to: $esphome_version"
    else
        echo "Failed to get ESPHome version from container"
    fi
fi
