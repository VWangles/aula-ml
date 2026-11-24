# aula-ml

Atividade Prática 07 — Versionamento de Dados e Modelos (UFC / CK0444).

O repositório versiona com Git os código, parâmetros, descritores DVC, manifestos, avaliações e tags. Os bytes dos CSVs e do artefato do modelo ficam no armazenamento DVC configurado à parte.

## Pré-requisitos

- Git (qualquer versão recente).
- Python 3.10 ou superior (foi usado Python 3.11.9).
- DVC 3: `pip install "dvc>=3,<4"`.
- Terminal Bash/Zsh. Neste ambiente (Windows sem WSL), os comandos DVC foram executados como `python3 -m dvc`.

Sem dependências externas para o treinamento: o classificador usa apenas a biblioteca padrão do Python.

## Estrutura

- `data/chamados.csv` — chamados sintéticos (`id`, `texto`, `classe`).
- `data/split.csv` — partição fixa (`treino` / `teste`).
- `data/manifesto.json` — origem, esquema, regra de rotulagem e histórico de alterações.
- `params.json` — parâmetros do tokenizador.
- `src/train.py` — treino, avaliação e registro de proveniência.
- `models/model.json` — artefato do modelo (rastreado pelo DVC).
- `reports/metrics.json` — acurácia e recall por classe.
- `reports/modelo.json` — manifesto do treino (hashes, commit, ambiente, `run_id`).
- `reports/comparacao.md` — comparação entre r1 e r2 e decisão da revisão.

## Armazenamento DVC

O remote padrão é local e relativo ao repositório:

```bash
dvc remote list
# storage ../dvc-storage   (default)
```

Por isso o clone de restauração deve ser criado como pasta irmã de `dvc-storage`. Para enviar e buscar os artefatos:

```bash
dvc push
dvc pull
```

Se o DVC reclamar de permissão no cache do sistema, defina `DVC_SITE_CACHE_DIR` para um diretório no seu perfil antes de executar os comandos.

## Treinamento

```bash
python3 src/train.py chamados-r1
python3 src/train.py chamados-r2
```

Cada execução sobrescreve `models/model.json`, `reports/metrics.json` e `reports/modelo.json`. Em seguida, use `dvc add models/model.json`, `git add` dos descritores/relatórios, `git commit` e `dvc push`.

## Versões marcadas

- `dados-v1` — snapshot inicial (com erro intencional no ID 006).
- `modelo-r1` — primeiro modelo, treinado em `chamados-v1`.
- `dados-v2` — rótulo do ID 006 corrigido para `urgente`.
- `modelo-r2` — modelo treinado em `chamados-v2`.

## Restaurando r1 em um clone limpo

A partir deste repositório:

```bash
URL_GITHUB=$(git remote get-url origin)
cd ..
git clone "$URL_GITHUB" aula-ml-restauracao
cd aula-ml-restauracao
git switch --detach modelo-r1
dvc pull
```

Antes do `dvc pull`, o clone contém apenas os descritores `.dvc`. Depois, confira os hashes:

```bash
python3 - <<'PY'
import hashlib
import json
from pathlib import Path

manifesto = json.loads(Path("reports/modelo.json").read_text(encoding="utf-8"))
if manifesto["model_id"] != "chamados-r1" or manifesto["dataset_id"] != "chamados-v1":
    raise SystemExit("Versão incorreta no manifesto.")
for caminho, campo in [
    ("data/chamados.csv", "dataset_sha256"),
    ("data/split.csv", "split_sha256"),
    ("models/model.json", "artifact_sha256"),
]:
    obtido = hashlib.sha256(Path(caminho).read_bytes()).hexdigest()
    if obtido != manifesto[campo]:
        raise SystemExit(f"Hash divergente para {caminho}")
    print(f"OK {caminho} {obtido}")
PY

git rev-parse HEAD
```

Se o `dvc pull` falhar, confirme `dvc remote list` no clone e se o `dvc push` da versão desejada terminou com sucesso.
