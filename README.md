> 🚧 **Projeto em andamento:** Agosto/2026. Fluxo executado até o **episódio 60**. Os episódios seguintes ainda serão baixados, verificados (`os.path.exists()`) e traduzidos com o Whisper (GPU T4) em novos lotes.

Por que este projeto existe: Este fluxo nasceu do pedido de um amigo que queria terminar de assistir a um anime da adolescência, disponível apenas no áudio original em Japonês. Como meu trabalho de legendagem atua no par Inglês-Português, testei uma solução com IA para gerar legendas em Inglês e tornar os episódios acessíveis a ele. :!: Nota: legendas geradas por IA facilitam o acompanhamento, mas não substituem a precisão de um nativo ou tradutor experiente em japonês.


# 🎬 Pipeline de Transcrição e Tradução de Animes

Automação em lote para download, verificação de integridade, gerenciamento de armazenamento em nuvem e geração de legendas traduzidas (Japonês ➔ Inglês) para episódios de anime hospedados no Internet Archive, utilizando **Agente de IA Gemini**, **Google Colab**, **yt-dlp** e **OpenAI Whisper**.

---

## 📋 Visão Geral

Este projeto resolve o gargalo de processamento e armazenamento na legendagem de séries longas (como acervos de 170+ episódios com média de 24 minutos cada). O fluxo divide a carga de trabalho entre ambientes de **CPU** (para operações leves de rede e download) e **GPU T4** (para inferência pesada de IA), contornando o limite de armazenamento gratuito do Google Drive por meio de uma rotação de lotes com o TeraBox.

## 🛠️ Tecnologias Utilizadas

* **Parceiro de Estudo, Código e Testes:** **Tech Partner** (agente de IA criado no Gemini apoiar a escrita, os testes e o acompanhamento do fluxo)
* **Ambiente de Execução:** Google Colaboratory (CPU & GPU NVIDIA T4)
* **Download de Mídia:** `yt-dlp`
* **Transcrição e Tradução (IA):** OpenAI `whisper` (modelo `medium`)
* **Linguagem & Automação:** Python 3 (`os`, loops de controle em lote)
* **Armazenamento:** Google Drive (área de trabalho) & TeraBox (arquivamento frio)

---

## 📂 Estrutura de Pastas e Arquivos

O pipeline padroniza os arquivos no Google Drive para evitar conflitos durante o processamento em lote:

```text
/content/drive/MyDrive/Episodios_Anime/
├── episodio_01.mp4       # Vídeo bruto baixado do Internet Archive
├── episodio_01.srt       # Legenda traduzida gerada pelo Whisper
├── episodio_02.mp4
├── episodio_02.srt
└── ...
⚙️ Arquitetura do Pipeline (Passo a Passo)
Fase 1: Coleta e Organização dos Links (Fonte)
Localiza-se a página do acervo no Internet Archive.
Utiliza-se a listagem de arquivos diretos (MPEG4 / Show All em Download Options) para extrair as URLs brutas dos arquivos .mp4. Isso evita falhas de redirecionamento ou erros de extração genérica no yt-dlp.
Fase 2: Download Automatizado (Ambiente CPU)
Execução via yt-dlp em um script Python com estrutura de repetição (for loop) sobre uma lista pré-definida de episódios.
Realiza o download direto para o Google Drive montado, gerenciando retentativas em caso de instabilidade de rede ou respostas HTTP Error 500 sem consumir cota de GPU.
Fase 3: Verificação de Segurança ("O Guardião da Porta")
Antes de registrar um episódio como concluído, o script executa uma validação física no diretório de destino utilizando os.path.exists().
Garante que o arquivo .mp4 foi efetivamente gravado no Google Drive, evitando falsos positivos caso o servidor da fonte interrompa a transferência.
Fase 4: Tradução por Inteligência Artificial (Ambiente GPU T4)
Com os vídeos validados no Drive, o ambiente do Colab é alternado para GPU T4.
O Whisper processa diretamente o arquivo .mp4, extraindo a faixa de áudio em japonês e gerando a legenda traduzida para o inglês (.srt):
Bash
whisper "/content/drive/MyDrive/Episodios_Anime/episodio_01.mp4" --model medium --language Japanese --task translate --output_format srt --output_dir "/content/drive/MyDrive/Episodios_Anime/"
Fase 5: Gestão de Armazenamento (Fluxo de Limpeza)
Para respeitar o limite de espaço gratuito do Google Drive, o pipeline opera em lotes de 4 a 5 episódios.
Após a geração e conferência dos arquivos .srt, os vídeos pesados (.mp4) são migrados para o TeraBox e excluídos do Google Drive, liberando espaço para o lote seguinte.
🚀 Como Executar
Monte o Google Drive no Google Colab e crie a pasta Episodios_Anime.
Configure o ambiente para CPU (Ambiente de execução > Alterar o tipo de ambiente de execução > CPU).
Instale o yt-dlp e execute o script de download em lote informando os links diretos coletados no Internet Archive.
Altere o ambiente para GPU T4 (Ambiente de execução > Alterar o tipo de ambiente de execução > GPU T4).
Instale o OpenAI Whisper (pip install -U openai-whisper) e rode o comando de tradução nos arquivos .mp4 salvos.
Faça o backup dos vídeos concluídos no TeraBox e limpe a pasta do Drive antes de iniciar o próximo lote.
📝 Licença
Este projeto é destinado a fins de estudo, preservação de acervos e acessibilidade audiovisual.
