# Load balancing considerations

We create a very unbalanced workload:

```julia-repl
julia> work_load = ceil.(Int, collect(10^3 * exp(-0.002*i) for i in 1:2^11));

julia> using UnicodePlots

julia> lineplot(work_load; xlabel="task", ylabel="workload", xlim=(1,2^11))
                  ┌────────────────────────────────────────┐ 
            1 000 │⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠘⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⢹⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⢳⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⢧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠈⢧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠀⠈⢳⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
   workload       │⠀⠀⠀⠀⠀⠀⠳⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠀⠀⠀⠀⠙⢦⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠀⠀⠀⠀⠀⠈⠳⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠳⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠙⢦⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠙⠲⢤⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                  │⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠓⠦⠤⣄⣀⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀│ 
                0 │⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠉⠓⠒⠒⠒⠦⠤⠤⠤⠤⠤⠤│ 
                  └────────────────────────────────────────┘ 
                  ⠀1⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀2 048⠀ 
                  ⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀task⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀ 
```

The scenario that we will consider below is the following: We want to parallelize the operation `sum(y -> log(y)^7, x)`, where `x` is a regular array. However,
to establish the uneven workload shown above, we will make each task sum up a different number of elements of `x`, specifically as many elements as is indicated by the `work_load` array for the given task/work item.

For parallelization, we will use `@spawn` and `@threads`, which, respectively, does and doesn't implement load balancing. We'll test those in conjunction with the chunking variants `BatchSplit()` and `ScatterSplit()` described above.

## Using `@threads`

First, we consider a variant where the `@threads` macro is used. The multithreaded operation is:

```julia-repl
julia> using Base.Threads, ChunkSplitters

julia> function uneven_workload_threads(x, work_load; n::Int, split::SplitStrategy)
           chunk_sums = Vector{eltype(x)}(undef, n)
           @threads for (ichunk, inds) in enumerate(chunks(work_load; n=n, split=split))
               local s = zero(eltype(x))
               for i in inds
                   s += sum(j -> log(x[j])^7, 1:work_load[i])
               end
               chunk_sums[ichunk] = s
           end
           return sum(chunk_sums)
       end
```

Using `n == Thread.nthreads() == 12`, we get the following timings:

```julia-repl
julia> using BenchmarkTools 

julia> @btime uneven_workload_threads($x, $work_load; n=nthreads(), split=BatchSplit())
  2.030 ms (71 allocations: 7.06 KiB)

julia> @btime uneven_workload_threads($x, $work_load; n=nthreads(), split=ScatterSplit())
  587.309 μs (70 allocations: 7.03 KiB)
```

Note that despite the fact that `@threads` doesn't balance load internally, one can get "poor man's load balancing" by using `ScatterSplit()` instead of `BatchSplit()`. This is due to the fact that for `ScatterSplit()` we create chunks by *sampling* from the entire workload: chunks will consist of work items with vastly different computational weight. In contrast, for `BatchSplit()`, the first couple of chunks will have high workload and the latter ones very low workload.

For `@threads`, increasing `n` beyond `nthreads()` typically isn't helpful. This is because it will anyways always create `nthreads()` tasks (i.e. a fixed number), grouping up multiple of our chunks if necessary.

```julia-repl
julia> @btime uneven_workload_threads($x, $work_load; n=8*nthreads(), split=BatchSplit());
  2.081 ms (74 allocations: 7.88 KiB)

julia> @btime uneven_workload_threads($x, $work_load; n=8*nthreads(), split=ScatterSplit());
  632.149 μs (75 allocations: 7.91 KiB)
```

## Using `@spawn`

We can use `@spawn` to get "proper" load balancing through Julia's task scheduler. The spawned tasks, each associated with a chunk of the `work_load` array, will be dynamically scheduled at runtime. If there are enough tasks/chunks, the scheduler can map them to Julia threads in such a way that the overall workload per Julia thread is balanced.

Here is the implementation that we'll consider.

```julia-repl
julia> function uneven_workload_spawn(x, work_load; n::Int, split::Symbol)
           ts = map(chunks(work_load; n=n, split=split)) do inds
               @spawn begin
                   local s = zero(eltype(x))
                   for i in inds
                       s += sum(log(x[j])^7 for j in 1:work_load[i])
                   end
                   s
               end
           end
           return sum(fetch.(ts))
       end
```

For `n == Thread.nthreads() == 12`, we expect to see similar performance as for the `@threads` variant above, because we're creating the same (number of) chunks/tasks.

```julia-repl
julia> @btime uneven_workload_spawn($x, $work_load; n=nthreads(), split=BatchSplit());
  1.997 ms (93 allocations: 7.30 KiB)

julia> @btime uneven_workload_spawn($x, $work_load; n=nthreads(), split=ScatterSplit());
  573.399 μs (91 allocations: 7.23 KiB)
```

However, by increasing `n > nthreads()` we can give the dynamic scheduler more tasks ("units of work") to balance out and improve the load balancing. In this case, the difference between `BatchSplit()` and `ScatterSplit()` chunking becomes negligible.

```julia-repl
julia> @btime uneven_workload_spawn($x, $work_load; n=8*nthreads(), split=BatchSplit());
  603.830 μs (597 allocations: 53.30 KiB)

julia> @btime uneven_workload_spawn($x, $work_load; n=8*nthreads(), split=ScatterSplit());
  601.519 μs (597 allocations: 53.30 KiB)
```
