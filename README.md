# Estudo — Fine-tuning de LLMs com Unsloth → GGUF → Ollama

> **Projeto para estudo**: demonstra o processo de **fine-tuning** de um LLM com **Unsloth (LoRA/QLoRA)**, teste de inferência, **exportação para GGUF** e **uso no Ollama**. Repositório inclui um notebook, um dataset de exemplo e um `Modelfile` para rodar localmente no **Ollama**.

---

## Objetivos do projeto
- Entender **quando** e **por que** fazer fine-tuning (vs. apenas “parameter tuning”).  
- Treinar com **Unsloth** com dados *input → output*.  
- **Exportar para GGUF** e **carregar no Ollama** via `Modelfile` para uso local.  

---

## Conteúdo
- `Fine Tuning.ipynb` — notebook de treino/inferência com Unsloth.  
- `json_extraction_dataset_500.json` — dataset de exemplo (HTML → JSON).  
- `Modelfile` — configuração para criar o modelo no **Ollama** a partir do `.gguf`.  

---

## Requisitos

### A) **Google Colab** (recomendado para o treino)
- GPU **T4** (Runtime → *Change runtime type* → GPU T4) e **Python 3**.
  
### B) **Ambiente local** (opcional)
- NVIDIA Driver + CUDA compatível (para treinar).  
- **Ollama** instalado (para rodar o `.gguf` localmente)
- Python **3.10+** para executar o notebook localmente (se preferir).

> Para **usar** o modelo no Ollama a CPU já tende a ser o suficiente, mas para o **treino** a GPU é melhor devido ao tempo.

---

## Passo a passo (Colab)

1. **Abrir o notebook**  
   Faça upload/abra `Fine Tuning.ipynb` no Colab e selecione **GPU T4**.

2. **Instalar dependências**  
   Execute as células iniciais do notebook para instalar `unsloth`, `transformers`, etc.

3. **Carregar o modelo base + tokenizer (Unsloth)**  
   Escolha um modelo pequeno para treinos rápidos (ajuste conforme seu caso).  
   A API do Unsloth segue o padrão `FastLanguageModel.from_pretrained(...)`.

4. **Preparar o dataset**  
   O projeto usa pares *input → output* (ex.: HTML → JSON).  
   Garanta que seu conjunto siga esse formato e seja tokenizável.

5. **Aplicar LoRA/QLoRA e treinar**  
   Configure o SFT Trainer (épocas, batch size, grad accumulation, max seq len, etc.) e inicie o treino.

6. **Testar a inferência (pós-treino)**  
   Rode alguns prompts de validação diretamente no notebook para verificar o comportamento.

7. **Exportar para GGUF**  
   Use a API nativa do Unsloth para exportar em **GGUF** (compatível com Ollama)

8. **Baixar o `.gguf`** para sua máquina.

---

## Carregar no Ollama (local)

1. **Coloque o `.gguf`** na mesma pasta do `Modelfile` deste repositório.

2. **Edite o `Modelfile`** (troque o nome do arquivo pelo seu `.gguf`):
   ```txt
   FROM ./seu-modelo-treinado.q4_k_m.gguf

   # Ajustes opcionais
   PARAMETER temperature 0.2
   PARAMETER top_p 0.9
   PARAMETER num_ctx 4096
   # Exemplos de stop tokens (se seu template exigir):
   # PARAMETER stop </s>

   SYSTEM Você é um assistente útil e conciso.
   ```

3. **Criar o modelo no Ollama**:
   ```bash
   ollama create meu-modelo -f Modelfile
   ollama list
   ```

4. **Rodar o modelo**:
   ```bash
   ollama run meu-modelo
   ```

5. **Exemplo de prompt**:
   ```text
   Extraia nome, preço, categoria e fabricante deste HTML: <div>...</div>
   ```
