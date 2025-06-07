# 🚨 Zabbix Mass Alert Detector (with Group or Tag Filtering)

Este projeto tem como objetivo **detectar automaticamente eventos massivos** no Zabbix, analisando os alertas ativos de forma inteligente por **grupo de hosts** ou, caso indisponível, pela **tag `CLIENTE`** configurada em cada host.

## 🧠 Motivação

Durante incidentes em larga escala, como falhas de rede ou quedas de serviços, é comum que **múltiplos hosts gerem alertas ao mesmo tempo**. Este script identifica esses casos e alerta o time de forma mais clara e centralizada.

Além disso, muitos ambientes possuem limitações de permissão na API do Zabbix, impedindo o retorno correto dos grupos. Por isso, o script usa **tags como alternativa** para identificar a origem dos hosts afetados.

---

## ⚙️ Funcionalidades

- Busca eventos ativos (`PROBLEM`) nos últimos _X_ minutos.
- Agrupa eventos por:
  - Grupo do host (`selectGroups`)
  - Tag `CLIENTE` (fallback caso o grupo não esteja disponível)
- Exibe:
  - Total de eventos por grupo ou cliente
  - Lista de hosts afetados
  - Detecta automaticamente se há **alertas massivos** com base em um número mínimo de ocorrências
- Tudo via **API do Zabbix**, sem necessidade de banco de dados ou interface

---

## 📥 Pré-requisitos

- Python 3.7+
- Acesso à API do Zabbix com token do tipo _Bearer Token_
- Biblioteca `requests` instalada

Instale com:

```bash
pip install requests
