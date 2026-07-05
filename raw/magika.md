---
url: https://github.com/google/magika
date_fetched: 2026-07-05
backfilled: true
---

Magika is a novel AI-powered file type detection tool that relies on the recent advance of deep learning to provide accurate detection. Under the hood, Magika employs a custom, highly optimized model that only weighs about a few MBs, and enables precise file identification within milliseconds, even when running on a single CPU. Magika has been trained and evaluated on a dataset of ~100M samples across 200+ content types (covering both binary and textual file formats), and it achieves an average ~99% accuracy on our test set.

Here is an example of what Magika command line output looks like:

Magika is used at scale to help improve Google users' safety by routing Gmail, Drive, and Safe Browsing files to the proper security and content policy scanners, processing hundreds billions samples on a weekly basis. Magika has also been integrated with VirusTotal (example) and abuse.ch (example).

For more context you can read our initial announcement post on Google's OSS blog, you can consult Magika's website, and you can read more in our research paper, published at the IEEE/ACM International Conference on Software Engineering (ICSE) 2025.

You can try Magika without installing anything by using our web demo, which runs locally in your browser!

- Available as a command line tool written in Rust, a Python API, and additional bindings for Rust, JavaScript/TypeScript (with an experimental npm package, which powers our web demo), and GoLang (WIP).
- Trained and evaluated on a dataset of ~100M files across 200+ content types.
- On our test set, Magika achieves ~99% average precision and recall, outperforming existing approaches -- especially on textual content types.
- After the model is loaded (which is a one-off overhead), the inference time is about 5ms per file, even when run on a single CPU.
- You can invoke Magika with even thousands of files at the same time. You can also use `-r`for recursively scanning a directory.
- Near-constant inference time, independently from the file size; Magika only uses a limited subset of the file's content.
- Magika uses a per-content-type threshold system that determines whether to "trust" the prediction for the model, or whether to return a generic label, such as "Generic text document" or "Unknown binary data".
- The tolerance to errors can be controlled via different prediction modes, such as `high-confidence`,`medium-confidence`, and`best-guess`.
- The client and the bindings are already open source, and more is coming soon!

Magika ships a CLI written in Rust, and can be installed in several ways.

Via `magika` python package:

`pipx install magika`Via brew (macOS / Linux)

`brew install magika`Via installer script:

`curl -LsSf https://securityresearch.google/magika/install.sh | sh`or:

`powershell -ExecutionPolicy Bypass -c "irm https://securityresearch.google/magika/install.ps1 | iex"`Via `magika-cli` Rust package:

`cargo install --locked magika-cli``pip install magika``npm install magika`Here you can find a number of quick examples just to get you started.

To learn about Magika's inner workings, see the Core Concepts section of Magika's website.

```
% cd tests_data/basic && magika -r * | head
asm/code.asm: Assembly (code)
batch/simple.bat: DOS batch file (code)
c/code.c: C source (code)
css/code.css: CSS source (code)
csv/magika_test.csv: CSV document (code)
dockerfile/Dockerfile: Dockerfile (code)
docx/doc.docx: Microsoft Word 2007+ document (document)
docx/magika_test.docx: Microsoft Word 2007+ document (document)
eml/sample.eml: RFC 822 mail (text)
empty/empty_file: Empty file (inode)
```
```
% magika ./tests_data/basic/python/code.py --json
[
  {
    "path": "./tests_data/basic/python/code.py",
    "result": {
      "status": "ok",
      "value": {
        "dl": {
          "description": "Python source",
          "extensions": [
            "py",
            "pyi"
          ],
          "group": "code",
          "is_text": true,
          "label": "python",
          "mime_type": "text/x-python"
        },
        "output": {
          "description": "Python source",
          "extensions": [
            "py",
            "pyi"
          ],
          "group": "code",
          "is_text": true,
          "label": "python",
          "mime_type": "text/x-python"
        },
        "score": 0.996999979019165
      }
    }
  }
]
```
```
% cat tests_data/basic/ini/doc.ini | magika -
-: INI configuration file (text)
```
```
% magika --help
Determines file content types using AI
Usage: magika [OPTIONS] [PATH]...
Arguments:
  [PATH]...
          List of paths to the files to analyze.
          Use a dash (-) to read from standard input (can only be used once).
Options:
  -r, --recursive
          Identifies files within directories instead of identifying the directory itself
      --no-dereference
          Identifies symbolic links as is instead of identifying their content by following them
      --colors
          Prints with colors regardless of terminal support
      --no-colors
          Prints without colors regardless of terminal support
  -s, --output-score
          Prints the prediction score in addition to the content type
  -i, --mime-type
          Prints the MIME type instead of the content type description
  -l, --label
          Prints a simple label instead of the content type description
      --json
          Prints in JSON format
      --jsonl
          Prints in JSONL format
      --format <CUSTOM>
          Prints using a custom format (use --help for details).
          The following placeholders are supported:
            %p  The file path
            %l  The unique label identifying the content type
            %d  The description of the content type
            %g  The group of the content type
            %m  The MIME type of the content type
            %e  Possible file extensions for the content type
            %s  The score of the content type for the file
            %S  The score of the content type for the file in percent
            %b  The model output if overruled (empty otherwise)
            %%  A literal %
  -h, --help
          Print help (see a summary with '-h')
  -V, --version
          Print version
```
For more examples and documentation about the CLI, see https://crates.io/crates/magika-cli.

```
>>> from magika import Magika
>>> m = Magika()
>>> res = m.identify_bytes(b'function log(msg) {console.log(msg);}')
>>> print(res.output.label)
javascript
```
```
>>> from magika import Magika
>>> m = Magika()
>>> res = m.identify_path('./tests_data/basic/ini/doc.ini')
>>> print(res.output.label)
ini
```
```
>>> from magika import Magika
>>> m = Magika()
>>> with open('./tests_data/basic/ini/doc.ini', 'rb') as f:
>>>     res = m.identify_stream(f)
>>> print(res.output.label)
ini
```
For more examples and documentation about the Python module, see the Python `Magika` module section.

Please consult Magika's website for detailed documentation about:

- Core Concepts
- How Magika works
- Models & content types
- Prediction modes
- Understanding the output
 
- CLI & Bindings (Python module, JavaScript module, ...)
- Contributing
- FAQ
- ...

Please contact us directly at magika-dev@google.com.

Apache 2.0; see `LICENSE` for details.

This project is not an official Google project. It is not supported by Google and Google specifically disclaims all warranties as to its quality, merchantability, or fitness for a particular purpose.
