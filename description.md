Hi,

I think the current example for `map_blocks(...)` has two issues:

1. In the current configuration it just works, and I think it does so for the wrong reason (see explanation below)
2. It falls a bit short of explaining how `map_blocks(...)` actually passes bits of data the supplied function, which I think it paramount to correctly apply the function (I added an example to the notebook to illustrate that the function could return an unexpected result if the user doesn't take care of settings chunks correctly)

What's the issue with the current example?

- Opening the dataset sets chunks on the time dimension
- `time_mean(obj)` actually computes the mean along the `lat` dimension (based on the function name and chunks on the time dimension, I guess this was not the intention (?) )
- The comparison with `.identical(...)` returns `True`, suggesting that everything worked fine and that this has to do with how the chunks on the time dimension are set in the example

```python
ds = xr.tutorial.open_dataset("air_temperature", chunks={"time": 100})

...

def time_mean(obj):
    # use xarray's convenient API here
    # you could convert to a pandas dataframe and use pandas' extensive API
    # or use .plot() and plt.savefig to save visualizations to disk in parallel.
    return obj.mean("lat")

# this will calculate values and will return True if the computation works as expected
ds.map_blocks(time_mean).identical(ds.mean("lat"))
```

The problem I see is: The comparison with `.identical(...)` works because actually the mean is computed along the lat dimension and the lat dimension is not chunked at all (more or less by chance), but it has nothing to do with setting the chunks of the time dimension when opening the dataset. Instead, when really computing the mean along the time dimension both in `time_mean(obj)` and via xarrays built-in `.mean("time")` the comparison via `.identical(...)` fails.

I have tried to update the notebook accordingly and add an exercise (now `Exercise 1`) to illustrate the importance of taking care of the chunks.

All the best
Andreas
