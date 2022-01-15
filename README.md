

## Run mkdocs in dev mode

Install virtualenv:
```console
apt install virtualenv
```

Clone the repository:
```console
git clone https://github.com/fusionSuite/documentation.git
cd documentation
```

Create a virtual environment:
```console
virtualenv venv
```

Activate your virtual environment:
```console
source venv/bin/activate
```

Install requirement for mkdocs
```console
pip install -r requirement.txt
```

Run development instance:
```console
mkdocs serve
```
