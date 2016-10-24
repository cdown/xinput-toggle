#!/bin/bash

debug() {
    # DEBUG comes from the environment
    if (( DEBUG )); then
        printf '%s\n' "$@" >&2
    fi
}

is_enabled() {
    local id="${1?}"
    enabled=$(
        xinput list-props "$id" |
            grep '\bDevice Enabled\b' | sed 's/.*\(.\)$/\1/'
    )
    # xinput returns 0 for disabled and 1 for enabled, so we invert since we
    # pass on 0
    return "$(( !enabled ))"
}

should_disable() {
    local id="${1?}"
    local force_enable="${2?}"
    local force_disable="${3?}"

    if (( force_enable )) && (( force_disable )); then
        echo '-d and -e make no sense together' >&2
        exit 3
    fi

    if (( force_enable )); then
        return 1
    elif (( force_disable )); then
        return 0
    elif is_enabled "$id"; then
        return 0
    else
        return 1
    fi
}

get_id_for_device_name() {
    local name="${1?}"
    xinput list "$name" | sed -n 's/.*id=\([0-9]\+\).*/\1/p'
}

show_help() {
    cat << EOF
Usage: ${0##*/} [-n]

Enable and disable xinput devices.

    -d          disable only, do not toggle
    -e          enable only, do not toggle
    -h          show this help page
    -i XID      only operate on device with xinput id XID
    -r REGEX    only operate on devices matching name REGEX
    -n          show results using notify-send in addition to stdout
    -p          print what we would do, but don't actually do it
    -t SECONDS  revert enable/disable after SECONDS seconds, ie. if you were
                enabling, after SECONDS seconds it will be disabled again.
                SECONDS must be an integer greater than 0
EOF
}

act_on_device() {
    local print="${1?}"
    local notify="${2?}"
    local timeout="${3?}"
    local xinput_action="${4?}"
    local our_next_flag="${5?}"
    local id="${6?}"

    if (( print )); then
        echo "Would $xinput_action device with id $id"
    else
        if ! msg="$(xinput -"$xinput_action" "$id" 2>&1)"; then
            printf '%s\n' "$msg" >&2
            notify-send "$msg"
            exit 6
        fi

        if (( notify )); then
            notify-send "${xinput_action^}d device with id $id"
        fi

        if (( timeout )); then
            debug "Added timeout $timeout"
            {
                sleep "$timeout"
                # TODO: Make args_without_timeour_or_force non-global (sigh,
                # shell makes this difficult due to pass by value)
                "$0" "${args_without_timeout_or_force[@]}" \
                    "$our_next_flag" -i "$id"
            } &
        fi
    fi
}

notify=0
force_enable=0
force_disable=0
timeout=0
print=0
only_xid=0
input_ids=()
args_without_timeout_or_force=()

while getopts dehi:npr:t: opt; do
    # We need to get a list of args without timeout/force for when we execute
    # timeout reversions. This is a blacklist of things we should not put in
    # the array, since they force enable/disable/name or do the timeout itself.
    case "$opt" in
        d|e|t|r)
            :  # Blacklisted
        ;;
        *)
            args_without_timeout_or_force+=( -"$opt" )
            if [[ $OPTARG ]]; then
                args_without_timeout_or_force+=( "$OPTARG" )
            fi
        ;;
    esac

    case "$opt" in
        'd') force_disable=1 ;;
        'e') force_enable=1 ;;
        'n') notify=1 ;;
        'i') only_xid="$OPTARG" ;;
        'h')
            show_help
            exit 0
        ;;
        'p') print=1 ;;
        'r') regex="$OPTARG" ;;
        't') timeout="$OPTARG" ;;
        '?')
            show_help >&2
            exit 1
        ;;
    esac
done

if (( only_xid )) && [[ "$regex" ]]; then
    echo '-r and -i cannot be currently used together' >&2
    exit 4
elif ! (( only_xid )) && ! [[ "$regex" ]]; then
    echo 'either -r or -i must be passed to filter devices' >&2
    exit 5
fi

if (( only_xid )); then
    input_ids=( "$only_xid" )
else
    mapfile -t matched_device_names < <(
        xinput list |
            sed -n 's/.*↳ \(.\+\)id=.*\[slave.*/\1/p' |
            sed 's/[\t ]*$//' |
            grep -i -- "$regex"
    )

    debug 'Matched device names:' "${matched_device_names[@]}"

    for name in "${matched_device_names[@]}"; do
        input_ids+=( "$(get_id_for_device_name "$name")" )
    done
fi

if (( "${#input_ids[@]}" == 0 )); then
    msg='No matching devices found (filters:'
    (( only_xid )) && msg+=" id=$only_xid"
    [[ $regex ]] && msg+=" regex=$regex"
    msg+=')'

    echo "$msg" >&2
    (( notify )) && notify-send "$msg"
    exit 2
fi

for id in "${input_ids[@]}"; do
    if should_disable "$id" "$force_enable" "$force_disable"; then
        act_on_device "$print" "$notify" "$timeout" disable -e "$id"
    else
        act_on_device "$print" "$notify" "$timeout" enable -d "$id"
    fi
done

# We may have queued some timeout reversion jobs
wait
