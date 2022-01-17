![alphafold.hegelab.org](imgs/af_hegelab.png)

# BetaFold

We ([hegelab.org](http://www.hegelab.org)) craeted this standalone AlphaFold (AlphaFold-Multimer, v2.1.1) fork with changes that most likely will not be inserted in the main repository, but we found these modifications very useful during our daily work. We plan to try to push these changes gradually to main repo via [our alphafold fork](https://github.com/hegelab/alphafold).

## Warning
* Currently, this is a no-Docker version. If you really need our functionalities inside a Docker Image, let us know.
* Earlier opction for the configuration file  was -c, now it is -C.

## Changes / Features

* It is called BetaFold, since there might be some minor bugs – we provide this code “as is”.
* This fork includes the **correction of memory issues** from [our alphafold fork](https://github.com/hegelab/alphafold) (listed below).
* The changes mostly affect the workflow logic.
* BetaFold run can be influence via **configuration files**.
* **Different steps of AF2 runs** (generating features; running models; performing relaxation) can be separated. Thus database searches can run on a CPU node, while model running can be performed on a GPU node. Note: timings.json file is overwritten upon consecutive partial runs – save it if you need it. 

## Configuration file

* You can provide the configuration file as: ‘run_alphafold.sh ARGUMENTS -C CONF_FILENAME’ (slightly modified version of the bash script from [AlfaFold without docker @ kalininalab](https://github.com/kalininalab/alphafold_non_docker); please see below our Requirement section)
* If no configuration file or no section or no option is provided, everything is expected to run everything with the original default parameters.

```
[steps]
get_features = true
run_models = true
run_relax = true

[relax]
top
```

## Requirements

* BetaFold uses the [AlfaFold without docker @ kalininalab](https://github.com/kalininalab/alphafold_non_docker) setup.

## Paper/Reference/Citation

Till we publish a methodological paper, please read and cite our preprint ["AlphaFold2 transmembrane protein structure prediction shines"](https://www.biorxiv.org/content/10.1101/2021.08.21.457196v1).

## Memory issues you may encounter when running original AlphaFold locally

### "Out of Memory"

This is expected to be included in the next AF2 release, see: [pull request #296](https://github.com/deepmind/alphafold/pull/296).

**Brief, somewhat outdated summary:** Some of our AF2 runs with short sequences (~250 a.a.) consumed all of our memory (96GB) and died. Our targets in these cases were highly conserved and produced a very large alignment file, which is read into the memory by a simple .read() in `alphafold/data/tools/jackhmmer.py` ` _query_chunk`. Importantly, the max_hit limit is applied at a later step to the full set, which resides already in the memory, so this option does not prevent this error.
* To overcome this issue exhausting the system RAM, we read the .sto file line-by-line, so only max_hit will reach the memory.
* Since the same data needed line-by-line for a3m conversion, we merged the two step together. We inserted to functions into `alphafold/data/parsers.py`: `get_sto` if only sto is needed and `get_sto_a3m` if also a3m is needed (the code is somewhat redundant but simple and clean).
* This issue was caused by `jackhmmer_uniref90_runner.query` and `jackhmmer_mgnify_runner.query`, so we modified the calls to this function in `alphafold/data/pipeline.py`.
* The called `query` in `alphafold/data/tools/jackhmmer.py` calls `_query_chunk`; from here we call our `get_sto*`; `_query_chunk` returns the `raw_output` dictionary, which also includes 'a3m' as a string or None.

## License and Disclaimer

Please see the [original](https://github.com/deepmind/alphafold#license-and-disclaimer).
