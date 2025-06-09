#!/usr/bin/env python3

import requests
import time
from collections import defaultdict

ZABBIX_API_URL = "http://x.x.x.x/api_jsonrpc.php"
ZABBIX_API_TOKEN = ""

headers = {
    "Content-Type": "application/json-rpc"
}

def api_request(method, params):
    payload = {
        "jsonrpc": "2.0",
        "method": method,
        "params": params,
        "id": 1
    }

    headers_with_auth = headers.copy()
    headers_with_auth["Authorization"] = f"Bearer {ZABBIX_API_TOKEN}"

    try:
        response = requests.post(ZABBIX_API_URL, json=payload, headers=headers_with_auth, timeout=10)
        return response.json()
    except Exception as e:
        print(f"❌ Error calling Zabbix API method {method}: {e}")
        return None

def detectar_massiva(min_alertas=5, intervalo_minutos=10):
    agora = int(time.time())
    inicio = agora - (intervalo_minutos * 60)

    params_event = {
        "output": ["eventid", "clock", "name", "value", "objectid"],
        "value": 1,
        "time_from": inicio,
        "sortfield": "clock",
        "sortorder": "DESC",
        "selectHosts": ["hostid", "name"],
        "limit": 1000
    }

    eventos_resp = api_request("event.get", params_event)
    if not eventos_resp or "error" in eventos_resp:
        print("❌ Error or empty response fetching events:", eventos_resp.get("error") if eventos_resp else "No response")
        return
    eventos = eventos_resp.get("result", [])

    if not eventos:
        print("No active events found.")
        return

    hostids = list({host["hostid"] for evento in eventos for host in evento.get("hosts", [])})
    if not hostids:
        print("No hosts associated with events.")
        return

    params_hosts = {
        "output": ["hostid", "name"],
        "selectGroups": ["groupid", "name"],
        "selectTags": "extend",
        "hostids": hostids
    }

    hosts_resp = api_request("host.get", params_hosts)
    if not hosts_resp or "error" in hosts_resp:
        print("❌ Error fetching hosts and groups:", hosts_resp.get("error") if hosts_resp else "No response")
        return
    hosts = hosts_resp.get("result", [])

    hostid_to_context = {}
    for h in hosts:
        hostid = h["hostid"]
        hostname = h["name"]
        groups = [g["name"] for g in h.get("groups", [])]
        tags = {t["tag"]: t["value"] for t in h.get("tags", [])}
        cliente_tag = tags.get("CLIENTE")

        if not groups and cliente_tag:
            groups = [cliente_tag]
        if not groups:
            continue

        hostid_to_context[hostid] = {
            "name": hostname,
            "groups": groups
        }

    group_event_count = defaultdict(int)

    for evento in eventos:
        for host in evento.get("hosts", []):
            hostid = host["hostid"]
            contexto = hostid_to_context.get(hostid)
            if not contexto:
                continue

            grupos = contexto["groups"]

            for grupo in grupos:
                group_event_count[grupo] += 1

    massiva_detectada = False
    for grupo, count in group_event_count.items():
        if count >= min_alertas:
            massiva_detectada = True
            print(f"MASSIVE ALERT DETECTED IN GROUP: {grupo}")
            # Apenas imprime o grupo com massiva, sem listar hosts

    if not massiva_detectada:
        print("No massive alerts detected.")

if __name__ == "__main__":
    detectar_massiva(min_alertas=5, intervalo_minutos=10)

