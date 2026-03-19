#!/bin/bash

# Folder with wallpapers
WALLPAPER_DIR="$HOME/Downloads/wallpaper"
CACHE_DIR="$HOME/.cache/wal-thumbnails"

# Grid dimensions
COLUMNS=3
LINES=3

# Create cache directory if it doesn't exist
mkdir -p "$CACHE_DIR"

# Generate thumbnail (bigger now, 200x200 for better stacking view)
generate_thumbnail() {
    local image="$1"
    local filename=$(basename "$image")
    local thumb="$CACHE_DIR/${filename}.png"
    if [ ! -f "$thumb" ]; then
        convert "$image" -resize 200x200^ -gravity center -extent 200x200 "$thumb" 2>/dev/null
    fi
    echo "$thumb"
}

# Build list of images with icons
entries=""
while IFS= read -r -d '' file; do
    thumb=$(generate_thumbnail "$file")
    name=$(basename "$file")
    entries+="$name\0icon\x1f$thumb\n"
done < <(find "$WALLPAPER_DIR" -maxdepth 1 -type f \( -iname "*.jpg" -o -iname "*.jpeg" -o -iname "*.png" -o -iname "*.bmp" -o -iname "*.gif" \) -print0 | sort -z)

# Rofi theme to stack icon above text
THEME="
element {
    orientation: vertical;
    padding: 10px;
}
element-icon {
    size: 2.5ch;
    padding: 5px;
}
element-text {
    padding: 5px;
    horizontal-align: 0.5;
}
"

SELECTED=$(echo -e "$entries" | rofi -dmenu -i -p "Wallpaper:" \
    -show-icons \
    -columns "$COLUMNS" \
    -lines "$LINES" \
    -theme-str "$THEME")

if [ -n "$SELECTED" ]; then
    IMAGE="$WALLPAPER_DIR/$SELECTED"
    if [ -f "$IMAGE" ]; then
        # 1. pywal mit dem neuen Bild ausführen (ohne Hintergrund zu setzen, -n)
        wal -n -i "$IMAGE"
        
        # 2. Hintergrund mit feh setzen
        feh --bg-fill "$IMAGE" --bg-fill "$IMAGE"
        
        # 3. Xresources neu laden (damit i3 die neuen Farben bekommt)
        xrdb -merge ~/.cache/wal/colors.Xresources
        
        # 4. i3 neu laden (damit die set_from_resource Direktiven aktualisiert werden)
        i3-msg reload
        
        killall -q polybar

        polybar sec &

        polybar main &
        
        # 5. Fertig! Kein Polybar, kein Dunst, keine Benachrichtigungen
        echo "Wallpaper und Farben wurden erfolgreich aktualisiert: $SELECTED"
    else
        echo "Fehler: Ausgewählte Datei nicht gefunden: $SELECTED"
    fi
fi
