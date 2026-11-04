# DL-Docking Studio v12.0

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Crispr9cas/DL-Docking-Studio/blob/main/Molecular_docking_DL_v12.ipynb)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23958930.svg)](https://doi.org/10.5281/zenodo.23958930)

## Visão geral

O **DL-Docking Studio** é um pipeline automatizado de docking molecular com *deep learning*, implementado em Python/Jupyter com interface Gradio e projetado para o Google Colab com GPU NVIDIA. Ele integra, em um único fluxo reprodutível:

- **Docking com rescoring 3D-CNN**: GNINA 1.3.2 (amostragem Monte Carlo com função empírica tipo AutoDock Vina e reavaliação por ensemble de redes convolucionais 3D: CNNscore, CNNaffinity, CNNvariance).
- **Auditoria físico-química das poses**: PoseBusters em modo `dock` e RMSD simétrico sem superposição prévia (spyrmsd) para validação por redocking.
- **Mapeamento de interações**: ProLIF restrito ao bolsão de 12 Å em torno do ligante, com contatos geométricos de contingência quando o ProLIF não conclui.
- **Ingestão nativa do AlphaFold DB**: download por código UniProt e cálculo do pLDDT do bolsão de ligação, com classe de confiabilidade.
- **Dinâmica molecular turnkey**: geração do script `run_md_openmm.py` (OpenMM, Amber14 + TIP3P-FB) para a pose líder.

Além disso, o pipeline preserva metais e cofatores catalíticos no receptor, executa benchmark frente ao ligante nativo (controle positivo) e exporta planilhas, gráficos de publicação e um relatório acadêmico (.docx, .pdf, .md, .tex, .bib).

> Escores de docking são estimativas computacionais. Não substituem medidas experimentais de afinidade, nem avaliações de ADME e toxicidade.

## Módulos técnicos

| Módulo | Ferramenta / método | Função no pipeline |
|---|---|---|
| Preparo do receptor | PDBFixer + OpenMM | Átomos ausentes, protonação em pH 7,4, variantes HID/HIE/CYX no sítio metálico, QC de protonação |
| Preparo do ligante | RDKit, OpenBabel | Dessalinização, ionização por regras, ETKDGv3, MMFF94, cargas de Gasteiger, TORSDOF |
| Docking e rescoring | GNINA 1.3.2 | Busca estocástica (semente 42), rescoring 3D-CNN, refino `--minimize` |
| Validação de poses | PoseBusters, spyrmsd | Testes oficiais do modo `dock`; RMSD simétrico no redocking |
| Interações | ProLIF (+ contatos geométricos) | Ligações de hidrogênio, π, hidrofóbicas, iônicas e metálicas no bolsão de 12 Å |
| Alvos preditos | AlphaFold DB | Download por UniProt (model_v4 → v3) e pLDDT do bolsão |
| Benchmark | Controle nativo | ΔVina, ΔpK, ΔLE e veredito multiparamétrico |
| Relatórios | openpyxl, matplotlib, python-docx, WeasyPrint | Planilhas, figuras (PNG 600 DPI/SVG/PDF), relatório acadêmico ABNT |
| Dinâmica molecular | OpenMM | Script `run_md_openmm.py` autônomo para a pose líder |

## Execução rápida no Google Colab

1. Clique no badge **Open in Colab** acima (ou abra `Molecular_docking_DL_v12.ipynb` no Colab).
2. Selecione um runtime com GPU (**Ambiente de execução → Alterar tipo de ambiente de execução → GPU**). Os testes de referência usaram uma NVIDIA A100.
3. Execute a célula única. Na primeira execução, o setup instala dependências e baixa o binário oficial do GNINA 1.3.2; reexecuções na mesma VM reaproveitam o que já está instalado.
4. Use a interface Gradio (link público gerado ao final) nos quatro passos: alvo e sítio ativo, ligante, docking, resultados e validação.

Recomenda-se uma cópia do notebook por usuário: o motor guarda estado global e processa uma operação por vez.

## Instalação local (pip/venv)

Requisitos: Python 3.10+, GPU NVIDIA com CUDA (recomendado), [OpenBabel](https://openbabel.org) (`obabel` no PATH) e o binário do [GNINA 1.3.2](https://github.com/gnina/gnina/releases/tag/v1.3.2) em `/usr/local/bin/gnina` (ou ajuste `GNINA_BIN` na célula).

```bash
git clone https://github.com/Crispr9cas/DL-Docking-Studio.git
cd DL-Docking-Studio
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
jupyter lab Molecular_docking_DL_v12.ipynb
```

A célula de setup foi escrita para o ambiente do Colab (apt, `/usr/local/bin`, caminhos CUDA). Em máquinas locais, instale os binários externos manualmente e revise esses trechos antes de executar.

## Como citar

**ABNT (NBR 10520 / NBR 6023):**

> SOUSA FILHO, Walter Paixão de. **DL-Docking Studio**: pipeline automatizado de docking molecular com rescoring 3D-CNN. Versão 12.0.0. Santa Maria: Universidade Franciscana, 2026. Disponível em: https://github.com/Crispr9cas/DL-Docking-Studio. DOI: 10.5281/zenodo.XXXXXXX.

Citação no texto: (SOUSA FILHO, 2026).

**BibTeX:**

```bibtex
@software{sousafilho2026dldocking,
  author    = {Sousa Filho, Walter Paix{\~a}o de},
  title     = {{DL-Docking Studio}: pipeline automatizado de docking molecular com rescoring 3D-CNN},
  version   = {12.0.0},
  year      = {2026},
  publisher = {Universidade Franciscana},
  url       = {https://github.com/Crispr9cas/DL-Docking-Studio},
  doi       = {10.5281/zenodo.XXXXXXX}
}
```

O arquivo [`CITATION.cff`](CITATION.cff) permite a citação direta pelo botão "Cite this repository" do GitHub. Ao usar os resultados, cite também as ferramentas subjacentes (GNINA, AutoDock Vina, PoseBusters, ProLIF, spyrmsd, RDKit, OpenMM, AlphaFold).

## Autoria e licença

- **Autor:** Walter Paixão de Sousa Filho, Universidade Franciscana (UFN), 2026.
- **Licença:** [MIT](LICENSE).
