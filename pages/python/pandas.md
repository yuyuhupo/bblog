## Pandas 函数（Function）

> 1、read_cvs()

    函数调用: info = pd.read_cvs(filename)
    函数功能: 读取指定的cvs文件，生产一个包含cvs数据的DataFrame
    传入参数: filename
        filename: str类型，需要读取的文件名
    返回参数: info
        info: DataFrame类型，读取文件生产的DataFrame
    类似方法还有: read_excel,read_json,read_sql,read_html等

> 2、isnull()

    函数调用: bool = pd.isnull(obj)
    函数功能: 返回一个包含数据是否是null的信息数据
    传入参数: obj
        obj: DataFrame/Series类型，待判断的数据
    返回参数: bool
        bool: DataFrame/Series类型，返回的判断结果，True表示null，False则不是

> 3、to_datetime()

    函数调用: date = pd.to_datetime(arg)
    函数功能: 将传入的数据转换成日期数据格式返回
    传入参数: arg
        arg: int/float/string/datetime/list/tuple/l-d array/Series类型，argument，可传入一维数组或Series，0.18.1版本中加入DataFrame和dict-like结构
    返回参数: date
        date: 返回的数据类型由传入的参数确定

    Note: pandas中通过to_datetime函数转换而成的数据其dtype为datetime64[ns],该数据存在的Series可以通过dt.month/year/day获取所需要的日期信息


## DataFrame类（class）

    类实例化: df = pd.DataFrame(data, index=)/pd.read_xxx(file_name)
    类的功能: 用于生成DateFrame
    传入参数: data, index/file_name
        data: ndarray类型，包含需要构建DataFrame的数据（二维）
        index: Series类型，决定作为索引的列参数
        file_name: str类型，需要读取的文件名
    返回参数: df
        df: DataFrame类型，读取文件生产的DataFrame

> 1、dtypes属性

    属性调用: fmt = df.dtypes
    属性功能: 返回数据结构中每列的数据类型（由于是多个，使用dtypes，numpy中单个使用dtype）
    属性参数: fmt
        fmt: Series类型，包含每个数据值的数据类型，index为列名，value为类型，其中，object类型相当于Python中的string


> 2、columns属性

    属性调用: index_name = pf.columns
    属性功能: 返回数据结构中每列的列名
    属性参数: index_name
        index_name: Index类型，<class 'pandas.core.indexes.base.Index' >，包含每列的列名

> 3、sharp属性方法

    属性调用: shp = df.shape
    属性功能: 返回数据结构的行列参数
    属性参数: shp 
        shp: tuple类型，（row, column），返回行列数

> 4、loc属性

    属性调用: index = df.loc
    属性功能: 返回一个index的类
    属性参数: index
        index: obj类型，<class 'pandas.core.indexing._LocIndexer'>，可用于切片获取数据信息的DataFrame，如index[0]获取第一行，index[3:7]获取3-7行的数据

> 5、head()方法

    函数调用: hdf = df.heag(num=5)
    函数功能: 返回cvs列表中的前num行数据
    传入参数: num
        num: int类型，需要获取的行数，默认为5
    返回参数: hdf
        hdf: DataFrame类型，原数据的前num行数据

> 6、tail()函数

    函数调用: tdf = df.tail(num=5)
    函数功能: 返回cvs列表中的后num行数据
    传入参数: num
        num: int类型，需要获取的行数
    返回参数: tdf
        tdf: DataFrame类型，原数据的后num行数据

> 7、describe()方法

    函数调用: ddf = df.describe()
    函数功能: 返回cvs列表中每个列的一些统计描述参数
    传入参数: 无
    返回参数: ddf
        ddf: DataFrame类型，包括的信息有：每一列的数量count，均值mean，标准差std，最小值min，1/4位数25%，中位数50%，3/4位数75%,最大值max

> 8、sort_values()方法

    函数调用: sdf = df.sort_values(by,axis=0,ascending=True,inplace=False,kind='quicksort',na_position='last')
    函数功能: 返回按参数排序后的DataFrame
    传入参数: by,axis,acsending,inplace,kind,na_position
        by: str类型，DateFrame的行列名
        axis: int类型，0按列sort，1按行sort
        ascending: bool类型，True为升序排列，False为降序排列
        inplace: bool类型，True则修改原DataFrame，False则返回新的DataFrame
        kind: str类型，确定后sort的排序算法，包括{'quicksort', 'mergesort', 'heapsort'}
        na_position: str类型，确定na数据存在的位置，'first'/'last'
    返回参数: sdf
        sdf: DataFrame类型，重排后的DataFrame

> 9、mean()方法

    函数调用: mdf = df.mean(axis=0)
    函数功能: 返回存储所有非NaN的值的平均值DataFrame
    传入参数: axis
        axis: int类型，0按列（第一轴）排序，1按行（最后轴）排序
    返回参数: mdf
        mdf: DataFrame类型，存储均值的数据类型为float

> 10、pivot_table()方法

    函数调用: cdf = df.pivo_table(index=,values=,aggfunc=)
    函数功能: 根据index将数据分组，对于values列的值（相同类型）执行aggfunc函数
    传入参数: index,values,aggfunc
        index: str类型，进行分组的列的别名
        values: str/list类型，需要计算的列的列名，多个则使用list
        aggfunc: method类型，需要调用的方法
    返回参数: cdf
        cdf: DataFrame类型，通过自定义函数运算后得到的DataFrame

> 11、dropna()方法

    函数调用: ddf = df.dropna(axis=0,how='any',thresh=None,subset=None,inplace=False)
    函数功能: 根据要求删除带有NaN值得行列
    传入参数: axis，how,thresh,subset,inplace
        axis: int/str类型，0/'index'为行搜索，1/'columns'为列搜索
        how: str类型，any只要出现NA值就删除该行/列数据，all所有值都是NA才删除
        thresh: int/None类型，表示对有效数据量的最小要求（为2则要求该行\列至少2个有效非NA数据存在）
        subset: str/list类型，表示在特定子集中寻找NA
        inplace: bool类型，表示是否在原数据上操作，True修改原数据，False返回新数据
    返回参数: ddf
        ddf: DataFrame类型，通过删除NA函数运算后得到的DataFrame

> 12、reset_index()方法

    函数调用: rdf = df.reset_index(level=None, drop=False, inplace=False, col_level=0, col_fill='')
    函数功能: 重置（一般为经过排序后的）DataFrame序号
    传入参数: level,drop,inplace,col_level,col_fill
        level: int/str/tuple/list类型，Only remove the given levels from the index. Removes all levels by default
        drop: bool类型，是否删除原始index列，True删除，False保留
        inplace: bool类型，是否在原数据上操作
        col_level: int/str类型， If the columns have multiple levels, determines which level the lables are  inserted into. By default it is inserted into first level.
        col_fill: obj类型，If the columns have multiple levels, determines how the other levels are named. If None then the index name is repeated.
    返回参数: rdf
        rdf: DataFrame 类型，通过重排index后的DataFrame

> 13、set_index()方法

    函数调用: sdf = df.set_index(keys, drop=True, append=False, inplace=False, verify_integrity=False)
    函数功能: 根据现有的columns参数重新设置index索引
    传入参数: keys,drop,append,inplace,verify_integrity
        keys: str类型，需要作为索引的列名
        drop: bool类型，是否删除作为索引的列，True删除，False保留
        append: bool类型，是否添加默认的index（序号索引）
        inplace: bool类型，是否在原数据上修改
        verify_integrity: bool类型，Check the new index of duplicates. Otherwise defer the check until necessary. Setting to False will improve the performance of this method
    返回参数: sdf
        sdf: DataFrame类型，通过重设index后的DataFrame

> 14、apply()方法

    函数调用: rdf = df.apply(func, axis=0, broadcast=False, raw=False, reduce=None, args=(), **kwds)
    函数功能: 对DataFrame相应的方向使用自定义的函数
    传入参数: func,axis,broadcast,raw,reduce,args,**kwds
        func: method类型，用于各行/列的函数
        axis: int/str类型，0/'index'对每行使用函数，1/'index'对每列使用函数
        broadcast: bool类型
        raw: bool类型
        reduce: bool/None类型
        args: tuple类型，Positional arguments to pass to function in addition to the array/series
        **kwds: 其余关键字参数将会被当做参数传给调用函数
    返回参数: rdf
        rdf: DataFrame类型，通过重排index后的DataFrame

> 15、ix属性

    属性调用: ix_obj = df.ix
    属性功能: 返回一个index类的数据
    属性参数: ix_obj
        ix_obj: obj类型，class 'pandas.core.indexing.IXIndexer'
        Note: 后续可通过ix_obj[rows, cols]获取DataFrame或Series， rows/cols可以是需要取的行索引/列名

## Series类

    类实例化: sr = pd.Series(data, index=)/df[column_name]
    类的功能: 用于生产Series
    传入参数: data,index/column_name
        data: ndarray类型，包含需要构建Series的数据（一维）
        index: Series类型，决定作为索引的列参数
        column_name: str类型，需要获取Series的列名
    返回参数: sr
        sr: Series类型，生产的Series
    
> 1、values属性

    属性调用: values = sr.values
    属性功能: 返回Series的所有值
    属性参数: values
        values: ndarray类型，Series的所有值形成的一维array

> 2、tolist()方法

    函数调用: list = sr.tolist()
    函数功能: 将Series或Index类的数据变成list形式返回
    传入参数: 无
    返回参数: list
        list: list类型，返回的数据列表

> 3、max/min()方法

    函数调用: value = sr.max/min()
    函数功能: 获取Series中的最大/最小值
    传入参数: 无
    返回参数: value
        value: int/str等类型，返回的最值

> 4、sort_values()方法

    函数调用: ssr = sr.sort_values(axis=0, ascending=True, inplace=False, kind='quicksort', na_position='last')
    函数功能: 返回按参数排序的Series
    传入参数: axis，ascending,inplace,kind,na_position
        axis: int类型，0按列（第一轴）排序，1按行（最后轴）排序
        ascending: bool类型，True为升序排列，False为降序排列
        inplace: bool类型，True则修改原Series，False则返回新的Series
        kind: str类型，确定sort的排序算法，包括{'quicksort', 'mergesort', 'heapsort'}
        na_position: str类型，确定na数据存在的位置，'first'/'last'
    返回参数: ssr
        ssr: Series类型，重排后的Series

> 5、mean()方法

    函数调用: msr = sr.mean()
    函数功能: 返回存储所有非NaN的值得平均值Series
    传入参数: 无
    返回参数: msr
        msr: Series类型，存储均值的数据类型为float

> 6、reset_index()方法

    函数调用: rsr = sr.reset_index(level=None,drop=False,name=None,inplace=False)
    函数功能: 重置（一般为经过排序后的）Series的序号
    传入参数: level,drop,name,inplace
        level: int/str/tuple/list类型，Only remove the given levels from the index. Removes all levels by default.
        drop: bool类型，是否删除原始的index列，True删除，False保留
        name: obj类型，The name of the column corresponding to the Series values
        inplace: bool类型，是否在原始数据上操作
    返回参数: rsr
        rsr: Series类型，通过重排index后的Series

> 7、values_counts()方法

    函数调用: csr = sr.values_counts(dropna=True)
    函数功能: 计算Series中各个values值的数量
    传入参数: dropna
        dropna: bool类型，是否计算NA的数量，True不计算，False计算
    返回参数: csr
        csr: Series类型，各数据值为索引，数量为value的Series

