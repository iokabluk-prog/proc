# proc
#!/bin/bash
echo "Показаны все открытые файлы"
printf "%-8s %-15s %-6s %s\n" "PID" "COMMAND" "FD" "FILE"
echo "----------------------------------------------------------------"

for pid in /proc/[0-9]*; do
    pid=${pid#/proc/}
    if [[ ! -d "/proc/$pid/fd" ]]; then
        continue
    fi

    # Получаем имя команды
    cmd=$(cat "/proc/$pid/comm" 2>/dev/null | cut -c1-15)
    [[ -z "$cmd" ]] && cmd="unknown"

    # Перебираем FD
    for fd in /proc/$pid/fd/*; do
        if [[ -L "$fd" ]]; then
            fd_num=${fd##*/}
            target=$(readlink "$fd" 2>/dev/null)
            if [[ -n "$target" ]]; then
                printf "%-8s %-15s %-6s %s\n" "$pid" "$cmd" "$fd_num" "$target"
            fi
        fi
    done 2>/dev/null
done
