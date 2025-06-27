# My useful Scripts
My usefule scripts

Docker Backup:
```bash
# Skrypt do backupu kontenerów Docker
# Autor: 19orzeszek90
# Data: $(date +%F)

#================== USTAWIENIA ==================
BACKUP_PATH="<path>" # dodaj scieże, do zapisywania kopii
TODAY=$(date +%F)
LOGFILE="${BACKUP_PATH}/docker_backup.log"

# Lista kontenerów do backupu (dodaj lub usuń wg potrzeby)
CONTAINERS=("Jellyfin" "Bazarr")

#================== FUNKCJE ==================

backup_container() {
    local name="$1"
    local container_name=$(echo "$name" | tr '[:upper:]' '[:lower:]')
    local backup_dir="${BACKUP_PATH}/${name}/backup_${TODAY}"
    local backup_file="backup_${container_name}.tar.gz"

    echo -e "\033[1;34m▶️  Zaczynam backup: $name\033[0m"
    sudo mkdir -p "$backup_dir"

    echo -e "   ⏳ Trwa backup $name..."
    sudo docker run --rm --volumes-from "$container_name" \
        -v "$backup_dir:/backup" busybox \
        tar czf "/backup/${backup_file}" /config \
        > /dev/null 2>> "$LOGFILE"

    if [ $? -eq 0 ]; then
        echo -e "\033[1;32m✅ Zakończono backup: $name\033[0m"
        echo -e "    Plik: $backup_dir/${backup_file}"
    else
        echo -e "\033[1;31m❌ Błąd podczas backupu: $name\033[0m"
        echo -e "    Sprawdź log: $LOGFILE"
    fi

    echo ""
}

#================== POWITANIE ==================

echo -e "\033[1;33m Witaj w skrypcie backupu kontenerów Docker!\033[0m"
echo -e " Data: $TODAY"
echo -e " Kontenery do backupu: ${CONTAINERS[*]}"
echo ""

#================== WYKONANIE ==================

for container in "${CONTAINERS[@]}"; do
    backup_container "$container"
done

echo -e "\033[1;36m✅ Wszystkie backupy wykonane.\033[0m"
echo -e " Log błędów (jeśli są): $LOGFILE"
echo -e ""

#================== USUWANIE STARYCH BACKUPÓW ==================

echo -e "\033[1;33m粒 Czyszczenie backupów starszych niż 7 dni...\033[0m"

for container in "${CONTAINERS[@]}"; do
    container_dir="${BACKUP_PATH}/${container}"

    # Szukamy i wypisujemy stare katalogi
    deleted_dirs=$(find "$container_dir" -maxdepth 1 -type d -name "backup_*" -mtime +7)

    if [ -n "$deleted_dirs" ]; then
        echo -e "\033[1;31m Usuwam stare backupy kontenera: $container\033[0m"
        echo "$deleted_dirs"
        # Usuwamy katalogi
        find "$container_dir" -maxdepth 1 -type d -name "backup_*" -mtime +7 -exec rm -rf {} \;
    else
        echo -e "✔️  Brak starych backupów dla: $container"
    fi
done
```
