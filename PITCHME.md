---

# @size[0.6em](Robust Data Processing Pipeline with Elixir and Flow)
## @size[0.6em](László Bácsi, 100Starlings)
### @size[0.6em](@fa[github]lackac @fa[twitter]@icanscale)

---

# Disclaimer

* first Elixir talk
* talk is based on uncompleted work
* based on client work<sup>*</sup>
* pipeline target is AWS Redshift

@css[small](<sup>*</sup> CollectPlus: parcel service in the UK)

---

## What does the machine do?

![what does the machine do?](assets/images/machine.png)

---

## Approaching a data project

![high level requirements](diagrams/pipeline_1.png)

+++

### Going Outside In

![one level down](diagrams/pipeline_2.png)

+++

### ETL, ELT, potayto, potahto

![ELT](diagrams/pipeline_3.png)

---

## Let It Flow

![Step 1](diagrams/step_1.png)

+++

## Glimpse of a Flow

``` elixir
def flow(input) do
  input
  |> Flow.from_enumerable(opts)
  |> Flow.filter(&feed_file?/1)
  |> Flow.map(&get_feed/1)
  |> Flow.map(&prepare_feed/1)
  |> Flow.map(&upload_feeds/1)
  |> Flow.partition(stages: 1, window: opts[:window])
  |> Flow.reduce(fn -> [] end, fn feed, acc -> [feed | acc] end)
  |> Flow.emit(:state)
end
```

@[2-3](Initialize the Flow from an enumerable)
@[4-7](Mapping stages prepare and upload the files in parallel)
@[8-9](Reduce feeds over a window into lists)

+++?image=assets/images/tubes.jpg&opacity=40

## It's a Series of Tubes

---

## Pipes, pipes, and more pipes

``` elixir
def split_feed({key, contents}) do
  feed_id = Path.basename(key, ".csv")

  contents
  |> String.splitter("\n")
  |> Stream.with_index()
  |> Stream.map(fn {line, i} ->
    Enum.join([String.trim(line), feed_id, i + 1], ",")
  end)
  |> Enum.reduce({"", ""}, fn
    @advice_row <> row, {advices, events} ->
      {advices <> row <> "\n", events}
    @event_row  <> row, {advices, events} ->
      {advices, events <> row <> "\n"}
    _, acc -> acc
  end)
end
```

@[4-5](Start with the full contents of the feeds and create a stream of lines)
@[6-9](Add feed_id and row number to each row)
@[10-16](Separate advices and events)

---
