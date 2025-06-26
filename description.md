Hi,

I think the current example for `map_blocks(...)` has two issues:

1. In the current configuration it just works, and I think it does so for the wrong reason
2. It falls a bit short of explaining how `map_blocks(...)` actually passes bits of data the supplied function

What's the issue with the current example?

- Opening the dataset sets chunks on the time dimension
- `time_mean(obj)` actually computes the mean along the `lat` dimension (based on the function name and chunks on the time dimension, I guess this was not the intention (?) )
- The comparison with `.identical(...)` returns `True`, suggesting that everything worked fine and that this has to do with how the chunks are set in the example

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

The problem I see is: The comparison with `.identical(...)` works because the lat dimension is not chunked, but it has nothing to do with setting the chunks of the time dimension when opening the dataset. Instead, when really computing the mean along the time dimension both in `time_mean(obj)` and via xarrays built-in `.mean("time")` the comparison via `.identical(...)` fails.
