# Step 2: Install Dependencies

Now that you've generated the module, let's navigate to its directory and install dependencies:

```bash
cd ../../connectors/source-<name>
poetry install
```


Let's verify everything is working as intended. Run:

```bash
poetry run source-<name> spec
```

You should see some output:

```text
{"type": "SPEC", "spec": {"documentationUrl": "https://docsurl.com", "connectionSpecification": {"$schema": "http://json-schema.org/draft-07/schema#", "title": "Python Http Tutorial Spec", "type": "object", "required": ["TODO"], "properties": {"TODO: This schema defines the configuration required for the source. This usually involves metadata such as database and/or authentication information.": {"type": "string", "description": "describe me"}}}}}
```

We just ran Airbyte Protocol's `spec` command! We'll talk more about this later, but this is a simple sanity check to make sure everything is wired up correctly.


## Notes on iteration cycle

### Dependencies

Python dependencies for your source should be declared in `airbyte-integrations/connectors/source-<source-name>/setup.py` in the `install_requires` field. You will notice that a couple of Airbyte dependencies are already declared there. Do not remove these; they give your source access to the helper interfaces provided by the generator.

You may notice that there is a `requirements.txt` in your source's directory as well. Don't edit this. It is autogenerated and used to provide Airbyte dependencies. All your dependencies should be declared in `setup.py`.

### Development Environment

The commands we ran above created a [Python virtual environment](https://docs.python.org/3/tutorial/venv.html) for your source. If you want your IDE to auto complete and resolve dependencies properly, point it at the virtual env `airbyte-integrations/connectors/source-<source-name>/.venv`. Also anytime you change the dependencies in the `setup.py` make sure to re-run `pip install -r requirements.txt`.

### Iterating on your implementation

There are two ways we recommend iterating on a source. Consider using whichever one matches your style.

**Run the source using python**

You'll notice in your source's directory that there is a python file called `main.py`. This file exists as convenience for development. You run it to test that your source works:

```bash
# from airbyte-integrations/connectors/source-<name>
poetry run source-<name> spec
poetry run source-<name> check --config secrets/config.json
poetry run source-<name> discover --config secrets/config.json
poetry run source-<name> read --config secrets/config.json --catalog sample_files/configured_catalog.json
```

The nice thing about this approach is that you can iterate completely within python. The downside is that you are not quite running your source as it will actually be run by Airbyte. Specifically, you're not running it from within the docker container that will house it.

**Run the source using docker**

If you want to run your source exactly as it will be run by Airbyte \(i.e. within a docker container\), you can use the following commands from the connector module directory \(`airbyte-integrations/connectors/source-python-http-example`\):

```bash
# First build the container
docker build . -t airbyte/source-<name>:dev

# Then use the following commands to run it
docker run --rm airbyte/source-<name>:dev spec
docker run --rm -v $(pwd)/secrets:/secrets airbyte/source-<name>:dev check --config /secrets/config.json
docker run --rm -v $(pwd)/secrets:/secrets airbyte/source-<name>:dev discover --config /secrets/config.json
docker run --rm -v $(pwd)/secrets:/secrets -v $(pwd)/sample_files:/sample_files airbyte/source-<name>:dev read --config /secrets/config.json --catalog /sample_files/configured_catalog.json
```

Note: Each time you make a change to your implementation you need to re-build the connector image via `docker build . -t airbyte/source-<name>:dev`. This ensures the new python code is added into the docker container.

The nice thing about this approach is that you are running your source exactly as it will be run by Airbyte. The tradeoff is iteration is slightly slower, as the connector is re-built between each change.
