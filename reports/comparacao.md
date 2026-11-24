# Comparação entre r1 e r2

## Alteração revisada

No dataset `chamados-v1`, o chamado `006` (`bloqueio de acesso`) estava rotulado como `comum`, contrariando a regra de rotulagem do manifesto (bloqueio é urgente). Em `dados-v2`, esse rótulo foi corrigido para `urgente` e o `dataset_id` passou a `chamados-v2`. O descritor `data/chamados.csv.dvc` mudou de hash (`0c11150657324c928472c835a49492b6` → `bd856dc82f2847a61b46c44c5da8d6ab`); o manifesto registra a explicação da mudança. A partição `data/split.csv` não foi alterada: `git diff dados-v1 dados-v2 -- data/split.csv.dvc` retorna vazio.

## Protocolo de avaliação

Mesma partição de teste nos dois treinos (`split_sha256 = 2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981`), com os IDs 007 a 010. Nenhuma seed é necessária: a regra de desempate é determinística.

| Modelo | Versão do dataset | Acurácia | Recall `comum` | Recall `urgente` |
|---|---|---|---|---|
| `chamados-r1` (`modelo-r1`) | `chamados-v1` | 3/4 (0.75) | 2/2 (1.0) | 1/2 (0.5) |
| `chamados-r2` (`modelo-r2`) | `chamados-v2` | 4/4 (1.0) | 2/2 (1.0) | 2/2 (1.0) |

O erro de `r1` ocorria porque o token `bloqueio` tinha um voto `comum` (ID 006, treino) e um voto `urgente` (ID 010, teste); com a correção, o token passou a ter dois votos `urgente` no treino e o chamado 010 passou a ser classificado corretamente.

## Evidência de restauração (clone limpo)

Clone de `https://github.com/VWangles/aula-ml.git` em pasta irmã de `dvc-storage`, com `git switch --detach modelo-r1` e `dvc pull`. Antes do `dvc pull`, apenas os descritores estavam presentes; depois, os três arquivos foram recuperados do armazenamento DVC e conferidos com o manifesto `reports/modelo.json`:

| Arquivo | Campo do manifesto | SHA-256 verificado |
|---|---|---|
| `data/chamados.csv` | `dataset_sha256` | `bc3fd86a3a5f598841a58839a9726d901fa7e8361ffe79c5b43873ad8a1328ac` |
| `data/split.csv` | `split_sha256` | `2ef820171c1a4625fe216e6da67c7c8dcf9d97a08288c0239cd23f1a6961d981` |
| `models/model.json` | `artifact_sha256` | `72a3a67936139a3473838a6dcfa7aae1acb4c23467be1ef1056d51d485b94840` |

Commit selecionado no clone: `61aab1caf3b88debe9e856e58948f335e33ee4b3` (`modelo-r1`). Os três hashes coincidem com os registrados no manifesto; nenhum treino foi necessário na restauração.

## Decisão

**Aceitar `r2` para este exercício.** Critério: melhorar o recall de urgentes sem reduzir o recall de comuns, mantendo a mesma partição de teste. O recall de urgentes subiu de 1/2 para 2/2 e o de comuns permaneceu 2/2; a acurácia subiu de 3/4 para 4/4. O `split_sha256` é idêntico, enquanto `dataset_sha256` e `artifact_sha256` mudaram, como esperado após a correção e o novo treino.

Ressalva: o conjunto de teste tem apenas quatro exemplos, portanto a diferença demonstra o efeito da correção neste exercício e não deve ser generalizada como estimativa de desempenho.
