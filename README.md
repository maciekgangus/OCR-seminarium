# OCR — od klasyki do modeli end-to-end

Notebook towarzyszący 45-minutowemu seminarium o OCR.

Pokazuje cały łuk technologiczny:
- klasyczny pipeline (binaryzacja, deskew, morfologia, segmentacja),
- klasyczne ML (HOG + SVM/kNN),
- mały CNN (LeNet w PyTorch),
- gotowe silniki: **Tesseract**, **EasyOCR**, **TrOCR**,
- porównanie ilościowe (CER) na tych samych obrazach.

---

## Szybki start (RHEL / Rocky / Alma 8/9)

```bash
# 1. Zależności systemowe
sudo dnf install -y python3.11 python3.11-pip python3.11-devel \
                    gcc gcc-c++ make \
                    mesa-libGL libglvnd-glx       # potrzebne dla opencv-python

# 2. Tesseract (EPEL trzeba mieć włączony)
sudo dnf install -y epel-release
sudo dnf install -y tesseract tesseract-langpack-pol tesseract-langpack-eng

# 3. Sklonuj repo i wejdź
git clone <REMOTE_URL> ocr-seminar
cd ocr-seminar

# 4. Wirtualne środowisko
python3.11 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip wheel

# 5. Zależności Python
pip install -r requirements.txt

# 6. Uruchom Jupytera
jupyter notebook ocr_seminar.ipynb
# albo bezgłowo:
# jupyter notebook --no-browser --ip=0.0.0.0 --port=8888
```

### Weryfikacja Tesseract

```bash
tesseract --version
tesseract --list-langs   # powinieneś zobaczyć "pol" i "eng"
```

### Alternatywa: konda (jeśli wolisz)

```bash
conda create -n ocr python=3.11 -y
conda activate ocr
conda install -c conda-forge tesseract poppler -y
pip install -r requirements.txt
```

---

## Co notebook pobiera przy 1. uruchomieniu

| Co | Rozmiar | Sekcja | Wyłączysz przez |
|----|---------|--------|-----------------|
| MNIST (torchvision) | ~12 MB | 5 | `ENABLE_CNN_TRAINING = False` |
| EasyOCR (english+detector) | ~70 MB | 6 | `ENABLE_EASYOCR = False` |
| TrOCR base printed (HF) | ~330 MB | 6 | `ENABLE_TROCR = False` |
| Obrazy testowe z GitHub | <5 MB | 1 | brak (jest fallback do generowania PIL-em) |

Cięższe sekcje mają flagi sterujące w komórce **Setup** — uruchom najpierw lekkie, dopiero potem włączaj kolejne.

## GPU (opcjonalnie)

Jeśli masz CUDA-capable GPU na RHEL:

```bash
# CUDA 12.1 wheel (sprawdź wersję NVIDIA driver: nvidia-smi)
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
```

EasyOCR i TrOCR automatycznie skorzystają z GPU jeśli `torch.cuda.is_available()`.

## Struktura katalogu po uruchomieniu

```
ocr-seminar/
  ocr_seminar.ipynb
  requirements.txt
  README.md
  .gitignore
  data/             # obrazy testowe (auto-pobrane lub wygenerowane)
  ~/.cache/         # modele HF/torch (cachowane systemowo)
```

## Troubleshooting

- **`ImportError: libGL.so.1`** → `sudo dnf install mesa-libGL` (opencv-python tego potrzebuje).
- **`TesseractNotFoundError`** → sprawdź `which tesseract`; jeśli puste, doinstaluj jak wyżej.
- **EasyOCR ściąga w nieskończoność** → pierwsze uruchomienie pobiera modele do `~/.EasyOCR/`; daj mu chwilę.
- **HF rate limit** → ustaw token: `huggingface-cli login` lub `export HF_TOKEN=...`.
- **Brak GPU, wolny TrOCR** → ustaw `ENABLE_TROCR = False` w sekcji 0; reszta zadziała na CPU.

## Licencja

MIT (materiały dydaktyczne, używaj śmiało).
