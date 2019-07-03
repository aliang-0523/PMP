由于是**kernel赛**，对dataFrame对象进行减小内存开销的操作<code>reduce_memory_usage</code>：操作具体过程如下:
```python
def reduce_mem_usage(df, verbose=True):
    numerics = ['int16', 'int32', 'int64', 'float16', 'float32', 'float64']
    start_mem = df.memory_usage().sum() / 1024**2    
    for col in df.columns:
        col_type = df[col].dtypes
        if col_type in numerics:
            c_min = df[col].min()
            c_max = df[col].max()
            if str(col_type)[:3] == 'int':
                if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                    df[col] = df[col].astype(np.int8)
                elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                    df[col] = df[col].astype(np.int16)
                elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                    df[col] = df[col].astype(np.int32)
                elif c_min > np.iinfo(np.int64).min and c_max < np.iinfo(np.int64).max:
                    df[col] = df[col].astype(np.int64)  
            else:
                if c_min > np.finfo(np.float16).min and c_max < np.finfo(np.float16).max:
                    df[col] = df[col].astype(np.float16)
                elif c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                    df[col] = df[col].astype(np.float32)
                else:
                    df[col] = df[col].astype(np.float64)    
    end_mem = df.memory_usage().sum() / 1024**2
    if verbose: print('Mem. usage decreased to {:5.2f} Mb ({:.1f}% reduction)'.format(end_mem, 100 * (start_mem - end_mem) / start_mem))
    return df
```
解读：利用<code>df.memory_usage()</code>统计开始时df内存开销，对df内每一列进行如下变换：统计每一列中的最大值<code>max</code>以及最小值<code>min</code>,若该列的最小值大于<code>int8</code>的最小值，并且该列的最大值小于<code>int8</code>的最大值，则将该列转换为<code>int8</code>,然后依次类推<code>int16</code>,<code>int32</code>,<code>int64</code>，找到匹配的区间。对于<code>float</code>来说:浮点类型的存储空间由大到小排列如下：<code>float64>float32>float16</code>
