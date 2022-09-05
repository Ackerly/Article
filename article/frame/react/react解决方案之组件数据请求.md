# react解决方案之组件数据请求
**普通数据请求**  
业务组件通常进行普通的数据请求，简单呈现获取到的数据，逻辑如下：  
1. 初始化data业务数据和loading状态值
2. mounted阶段发起请求获得业务数据，并在在请求的过程中同步loading状态值
3. 根据loading状态渲染data业务数据

示例代码如下：
```
import React, { useEffect, useState } from "react";
import { Spin } from "@arco-design/web-react";

const Cmpt: React.FC = () => {
  // 1. 初始化data业务数据和loading状态值
  const [data, setData] = useState("");
  const [loading, setLoading] = useState(false);

  // 2. mounted阶段发起请求获得业务数据，并在在请求的过程中同步loading状态值
  useEffect(() => {
    setLoading(true);
    fetchData()
      .then(setData)
      .finally(() => setLoading(false));
  }, []);

  // 3. 根据loading状态渲染data业务数据
  return <Spin loading={loading}>{data}</Spin>;
};

export default Cmpt;
```
重复写loading和data变量挺烦的，可以将这段数据请求与状态更新逻辑抽离成一个公共的数据请求Hook。实现如下：
```
import React from 'react';
import { debounce } from 'lodash';

export interface IBuildUseFetchOptions<T, P> {
  /** 是否立即查询，默认值为true */
  immediate?: boolean;
  /** 防抖间隔（毫秒），默认值为300 */
  duration?: number;
  /** 关联属性，默认值为空数组 */
  relation?: Array<string>;
  /** 筛选条件，默认值为空数组 */
  properties?: Array<string | { key: string; value: any }>;
  /** 筛选条件转换钩子函数 */
  getQuery?: (query: any, props: P) => any;
  /** 加载数据钩子函数 */
  getData?: (query: any, props: P) => Promise<T>;
  /** 自定义错误逻辑 */
  onError?: (err: any) => void;
}

/**
 * 数据请求Hook工厂函数
 * @param options 数据请求Hook配置
 * @returns
 */
export function buildUseFetch<T, P = {}>(options: IBuildUseFetchOptions<T, P>) {
  const {
    immediate = true,
    duration = 200,
    relation = [],
    properties = [],
    getQuery = query => query,
    getData = () => Promise.resolve(undefined),
    onError = err => console.error(err),
  } = options;

  return function useFetch<C = {}>(props: P, defaultQuery?: C) {
    const [inited, setInited] = React.useState(false);
    const [loading, setLoading] = React.useState(immediate);

    const [data, setData] = React.useState<T>();

    const [query, setQuery] = React.useState({
      ...properties.reduce((p, c) => {
        if (typeof c === 'object') {
          p[c.key] = c.value;
        } else {
          p[c] = undefined;
        }
        return p;
      }, {} as any),
      ...(defaultQuery || {}),
    });

    // 缓存变量
    const ref = React.useRef({
      props,
      inited,
      loading,
      query,
      data,
      isUnmounted: false,
    });

    // 数据请求方法
    const onFetch = React.useCallback(
      debounce(_query => {
        setTimeout(async () => {
          try {
            const newQuery = getQuery(
              {
                ..._query,
              },
              ref.current.props
            );

            const targetQuery = Object.keys(newQuery).reduce((p, c) => {
              const v = newQuery[c];
              if (v !== undefined && v !== null) {
                p[c] = v;
              }
              return p;
            }, {} as any);

            const _data = await getData(targetQuery, ref.current.props);

            !ref.current.isUnmounted && setData(_data);
            ref.current.data = _data;
          } catch (err) {
            onError(err);
          } finally {
            !ref.current.isUnmounted && setLoading(false);
            ref.current.loading = false;
          }
        });
      }, duration),
      []
    );

    // 数据加载方法
    const onLoad = React.useCallback(_query => {
      setLoading(true);
      ref.current.loading = true;

      onFetch(_query);
    }, []);

    // 数据刷新方法
    const onRefresh = React.useCallback(() => {
      onLoad(ref.current.query);
    }, []);

    // 组件更新时监测查询参数变更，若变更自动执行数据加载方法
    React.useEffect(() => {
      if (!ref.current.inited) {
        return;
      }
      ref.current.query = query;
      onRefresh();
    }, [query]);

    // 组件更新时监测组件Props参数变更（通过关联属性过滤），若变更自动执行数据加载方法
    React.useEffect(() => {
      if (!ref.current.inited) {
        return;
      }
      const oldProps = ref.current.props;
      ref.current.props = props;
      if (relation.find(p => (oldProps as any)[p] !== (props as any)[p])) {
        onRefresh();
      }
    }, [props]);

    // 组件初始化时判断是否自动执行数据加载方法
    React.useEffect(() => {
      setInited(true);
      ref.current.inited = true;

      if (immediate) {
        onLoad(ref.current.query);
      }

      return () => {
        ref.current.isUnmounted = true;
      };
    }, []);

    const fetchResult = {
      inited,
      loading,
      query,
      data,
    };

    const fetchAction = {
      setInited,
      setLoading,
      setQuery,
      setData,
      onLoad,
      onRefresh,
    };

    const ret: [typeof fetchResult, typeof fetchAction] = [
      fetchResult,
      fetchAction,
    ];

    return ret;
  };
}
```
这个数据请求Hook来优化之前繁冗的代码，优化后结果如下：
```
import React from "react";
import { Spin } from "@arco-design/web-react";

// 通过工厂函数生成Hook函数方便复用
const useFetch = buildUseFetch({
  // 数据请求逻辑
  getData: fetchData,
});

const Cmpt: React.FC = (props) => {
  // 使用数据请求Hook
  const [{ loading, data }] = useFetch(props);

  return <Spin loading={loading}>{data}</Spin>;
};

export default Cmpt;
```
**自定义请求参数与参数转换**  
业务系统中最常见的一个场景，点击「查看按钮」跳转详情页面，这时候就需要通过获取「路径参数」发起「数据详情的请求」。
「useFetch」支持自定义请求参数来应对这种请求场景，示例代码如下：  
```
import React from "react";
import { useParams } from "react-router";

const useFetch = buildUseFetch({
  // 数据请求逻辑
  getData: (query) => fetchData(query.id),
});

const Cmpt: React.FC = (props) => {
  // 获取路由参数
  const { id } = useParams<{ id: string }>();

  // 传入请求参数
  const [{ data }] = useFetch(props, {
    id,
  });

  // ... 略
};

export default Cmpt;
```
如果请求参数需要转换，「useFetch」同样支持在「build」过程中定义「参数转换」逻辑，如下：
```
import React from "react";
import { useParams } from "react-router";

const useFetch = buildUseFetch({
  // 请求参数转换逻辑
  getQuery: ({ id }) => {
    return {
      id,
      timestamp: new Date().getTime(),
    };
  },
  // 数据请求逻辑
  getData: ({ id, timestamp }) => fetchData(id, timestamp),
});

const Cmpt: React.FC = (props) => {
  // 获取路由参数
  const params = useParams<{ id: string }>();

  // 传入请求参数
  const [{ data }] = useFetch(props, params);

  // ... 略
};

export default Cmpt;
```
**手动触发请求**  
业务场景中通常存在这么一个场景，组件「不自行发起请求」，待用户「执行动作确认请求参数」后再「发起请求」。  
「useFetch」默认「立即」发起请求，但在「build」的时候支持设定「不立即」发起请求，示例代码如下：
```
import React from "react";
import { Button } from "@arco-design/web-react";

const useFetch = buildUseFetch({
  // 不立即请求
  immediate: false,
  // 数据请求逻辑
  getData: fetchData,
});

const Cmpt: React.FC = (props) => {
  const [{ data }, { onLoad }] = useFetch(props);

  return (
    <>
      {/* ... 略 */}

      {data}

      {/* 用户自行触发数据请求 */}
      <Button onClick={() => onLoad({ id: "a" })}>A</Button>
      <Button onClick={() => onLoad({ id: "b" })}>B</Button>
    </>
  );
};

export default Cmpt;
```
数据Hook每传入一个请求参数都会作为最终的请求参数缓存起来，因此想复用上一次请求的参数重新发起数据请求就可以使用「onRefresh」，示例代码如下：  
```
// ... 略

  const [{ data }, { onRefresh }] = useFetch(props);
  
  return (
    <>
      {/* ... 略 */}

      {data}

      {/* 重新触发数据请求 */}
      <Button onClick={onRefresh}>Refresh</Button>
    </>
  );

// ... 略
```
**监听组件属性变化**  
存在这么一种业务组件，它的某种属性变化，需要将该属性作为请求参数重新发起请求并将请求结果进行呈现。  
用过Vue开发的读者们肯定对其「watch」能力喜爱有加，React同样可以做到「watch」。  
React并非直接「watch」，而是「props」或「state」的变化会触发「update」，通过「新的props或state」与「旧的props或state」进行比对判断某个属性或状态是否变化，从而执行某个动作，实现watch的效果。  
对于这种场景，「useFetch」的使用示例代码如下：  
```
import React from "react";

type IProps = {
  id: string;
};

const useFetch = buildUseFetch<unknown, IProps>({
  // 设定监听的属性名，被监听的属性值变化会触发重新请求
  relation: ["id"],
  // 使用属性作为请求参数
  getData: (_, props) => fetchData(props.id),
});

const Cmpt: React.FC<IProps> = (props) => {
  const [{ data }] = useFetch(props);

  return <>{data}</>;
};

export default Cmpt;
```
## 列表数据请求
列表数据请求是数据请求细分出的一类常见的请求场景，页面流程与普通数据请求流程相似。它与普通请求的不同点在于列表请求不仅在mounted阶段请求，例如：  
- 提供列表相关分页属性（当前分页页码、总页数和是否有下一页等）
- 提供列表相关操作接口（刷新和下一页等）
- 关联查询条件（查询条件的变更会触发重新请求）

示例代码如下：  
```
import React, { useEffect, useState } from "react";
import { Table, Input } from "@arco-design/web-react";

const Cmpt: React.FC = () => {
  // 初始化列表相关状态
  const [data, setData] = useState([]);
  const [query, setQuery] = useState<Record<string, any>>({
    pageNo: 1,
    pageSize: 10,
  });
  const [total, setTotal] = useState(0);
  const [loading, setLoading] = useState(false);

  // 列表数据请求方法
  const fetchData = (query: Record<string, any>) => {
    setQuery(query);
    setLoading(true);

    fetchData(query)
      .then((res) => {
        setData(res.data);
        setTotal(res.total);
      })
      .finally(() => setLoading(false));
  };

  // mounted发起请求初始化列表数据
  useEffect(() => {
    fetchData(query);
  }, []);

  return (
    <>
      <Input
        onChange={(v) => {
          // 手动触发重新请求
          const newQuery = { ...query, keyword: v };
          fetchData(newQuery);
        }}
      />
      <Table
        loading={loading}
        data={data}
        pagination={{
          current: query.pageNo,
          pageSize: query.pageSize,
          total,
          onChange: (current, size) => {
            // 手动触发重新请求
            fetchData({
              pageNo: current,
              pageSize: size,
            });
          },
        }}
      />
    </>
  );
};

export default Cmpt;
```
跟优化普通数据请求方法一致，我们可以可以将这段数据请求与状态更新逻辑抽离成一个公共的列表请求Hook，实现如下：
```
import React from 'react';
import { debounce, omit } from 'lodash';

/** 平台标识 */
export enum EListPlatform {
  /** 桌面端 */
  'Desktop' = 'Desktop',
  /** 移动端 */
  'Mobile' = 'Mobile',
}

export type IUseListData<T> = Record<string, unknown> & {
  /** 数据 */
  data: Array<T>;
  /** 数据总量 */
  totalSize: number;
};

export type IUseListQuery = {
  /** 分页页码 */
  pageNo: number;
  /** 分页大小 */
  pageSize: number;
};

export interface IBuildUseListOptions<T, P> {
  /** 平台标识，默认值为Desktop */
  platform?: EListPlatform;
  /** 是否立即查询，默认值为true */
  immediate?: boolean;
  /** 防抖间隔（毫秒），默认值为300 */
  duration?: number;
  /** 关联属性，默认值为空数组 */
  relation?: Array<string>;
  /** 筛选条件，默认值为空数组 */
  properties?: Array<string | { key: string; value: any }>;
  /** 筛选条件转换钩子函数 */
  getQuery?: (query: any, props: P) => any;
  /** 加载数据钩子函数 */
  getData?: (query: any, props: P) => Promise<IUseListData<T>>;
  /** 自定义错误逻辑 */
  onError?: (err: any) => void;
}

/**
 * 列表请求Hook工厂函数
 * @param options 列表请求Hook配置
 * @returns
 */
export function buildUseList<T, P = {}>(options: IBuildUseListOptions<T, P>) {
  const {
    // platform = EListPlatform.Desktop,
    immediate = true,
    duration = 200,
    relation = [],
    properties = [],
    getQuery = query => query,
    getData = () => Promise.resolve({ data: [], totalSize: 0 }),
    onError = err => console.error(err),
  } = options;

  return function useList<C = {}>(
    props: P,
    _defaultQuery?: C & Partial<IUseListQuery>
  ) {
    const [inited, setInited] = React.useState(false);
    const [loading, setLoading] = React.useState(immediate);
    const [loadingMore, setLoadingMore] = React.useState(false);

    const defaultQuery = React.useMemo(
      () => ({
        pageNo: 1,
        pageSize: 10,
        ..._defaultQuery,
      }),
      []
    );
    const [pageNo, setPageNo] = React.useState(defaultQuery.pageNo);
    const [pageSize, setPageSize] = React.useState(defaultQuery.pageSize);
    const [totalSize, setTotalSize] = React.useState(0);

    const [list, setList] = React.useState<Array<T>>([]);
    const [data, setData] = React.useState<IUseListData<T>>({
      data: list,
      totalSize: totalSize,
    });

    const [query, setQuery] = React.useState({
      ...properties.reduce((p, c) => {
        if (typeof c === 'object') {
          p[c.key] = c.value;
        } else {
          p[c] = undefined;
        }
        return p;
      }, {} as any),
      ...omit(defaultQuery, ['pageNo', 'pageSize']),
    });

    // 是否允许加载更多
    const hasMore = React.useMemo(
      () => pageSize * pageNo < totalSize && list.length < totalSize,
      [pageSize, pageNo, list, totalSize]
    );

    // 缓存变量
    const ref = React.useRef({
      props,
      pageNo,
      pageSize,
      totalSize,
      inited,
      loading,
      loadingMore,
      query,
      data,
      list,
      isUnmounted: false,
    });

    // 加载中状态变更方法
    const changeLoading = React.useCallback(
      (active?: boolean, more?: boolean) => {
        if (active) {
          setLoading(true);
          ref.current.loading = true;
          if (more) {
            setLoadingMore(true);
            ref.current.loadingMore = true;
          }
        } else {
          setLoading(false);
          ref.current.loading = false;
          setLoadingMore(false);
          ref.current.loadingMore = false;
        }
      },
      []
    );

    // 数据请求方法
    const onFetch = React.useCallback(
      debounce(_query => {
        setTimeout(async () => {
          try {
            // 通过筛选条件转换钩子函数获取转换后的请求条件
            const newQuery = getQuery(
              {
                pageNo: ref.current.pageNo,
                pageSize: ref.current.pageSize,
                ..._query,
              },
              ref.current.props
            );

            // 过滤值为undefined或null的请求条件
            const targetQuery = Object.keys(newQuery).reduce((p, c) => {
              const v = newQuery[c];
              if (v !== undefined && v !== null) {
                p[c] = v;
              }
              return p;
            }, {} as any);

            const result = await getData(targetQuery, ref.current.props);

            !ref.current.isUnmounted && setData(result);
            ref.current.data = result;

            const { data = [], totalSize: _totalSize = 0 } = result;

            !ref.current.isUnmounted && setTotalSize(_totalSize);
            ref.current.totalSize = _totalSize;

            let _list: Array<T> = [];
            if (ref.current.loadingMore) {
              _list = ref.current.list.concat(data);
            } else {
              _list = data;
            }
            !ref.current.isUnmounted && setList(_list);
            ref.current.list = _list;
          } catch (err) {
            onError(err);
          } finally {
            !ref.current.isUnmounted && changeLoading(false);
          }
        });
      }, duration),
      []
    );

    // 数据加载方法
    const onLoad = React.useCallback(
      (_query, _options?: { more?: boolean }) => {
        changeLoading(true, _options?.more);
        onFetch(_query);
      },
      []
    );

    // 数据刷新方法
    const onRefresh = React.useCallback((reload?: boolean) => {
      const _reload = reload === undefined ? true : reload;
      if (_reload) {
        setPageNo(1);
        ref.current.pageNo = 1;
      }

      onLoad(ref.current.query);
    }, []);

    // 数据加载更多方法
    const onLoadMore = React.useCallback(
      debounce(() => {
        changeLoading(true, true);

        setPageNo(ref.current.pageNo + 1);
        ref.current.pageNo++;
      }, duration),
      []
    );

    // 组件更新时监测页码变更，若变更自动执行数据加载方法
    React.useEffect(() => {
      if (!ref.current.inited) {
        return;
      }
      ref.current.pageNo = pageNo;
      onLoad(ref.current.query);
    }, [pageNo]);

    // 组件更新时监测页数和查询参数变更，若变更自动执行数据加载方法
    React.useEffect(() => {
      if (!ref.current.inited) {
        return;
      }
      ref.current.pageSize = pageSize;
      ref.current.query = query;
      onRefresh(true);
    }, [pageSize, query]);

    // 组件更新时监测组件Props参数变更（通过关联属性过滤），若变更自动执行数据加载方法
    React.useEffect(() => {
      if (!ref.current.inited) {
        return;
      }
      const oldProps = ref.current.props;
      ref.current.props = props;
      if (relation.find(p => (oldProps as any)[p] !== (props as any)[p])) {
        onRefresh(true);
      }
    }, [props]);

    // 组件初始化时判断是否自动执行数据加载方法
    React.useEffect(() => {
      setInited(true);
      ref.current.inited = true;

      if (immediate) {
        onLoad(ref.current.query);
      }

      return () => {
        ref.current.isUnmounted = true;
      };
    }, []);

    const listResult = {
      inited,
      loading,
      loadingMore,
      pageNo,
      pageSize,
      totalSize,
      hasMore,
      query,
      list,
      data,
    };
    const listAction = {
      setInited,
      setLoading,
      setLoadingMore,
      setPageNo,
      setPageSize,
      setTotalSize,
      setQuery,
      setList,
      setData,
      onLoad,
      onRefresh,
      onLoadMore,
    };

    const ret: [typeof listResult, typeof listAction] = [
      listResult,
      listAction,
    ];

    return ret;
  };
}
```
用这个列表请求Hook来优化之前繁冗的代码，优化后结果如下：
```
import React from "react";
import { Table, Input } from "@arco-design/web-react";

const useList = buildUseList({
  getData: fetchData,
});

const Cmpt: React.FC = (props) => {
  // 初始化列表相关状态
  const [
    { loading, query, list, pageNo, pageSize, total },
    { setPageNo, setPageSize, setQuery },
  ] = useList(props);

  return (
    <>
      <Input
        onChange={(v) => {
          // 查询条件的变更会自动触发重新请求
          const newQuery = { ...query, keyword: v };
          setQuery(newQuery);
        }}
      />
      <Table
        loading={loading}
        data={list}
        pagination={{
          current: pageNo,
          pageSize: pageSize,
          total,
          onChange: (current, size) => {
            // 查询条件的变更会自动触发重新请求
            setPageNo(current);
            setPageSize(size);
          },
        }}
      />
    </>
  );
};

export default Cmpt;
```
useList」关联查询条件，查询条件的变更会触发重新请求，这里的查询条件包含：pageNo、pageSize、query。  
因此setPageNo、setPageSize、setQuery都会触发数据请求。  
**列表刷新请求**  
业务场景中存在数据操作或列表操作的场景：  
- 数据操作有「编辑」数据等，这类操作成功后需要刷新列表，页码不变；
- 一般列表操作有「新增」数据等，这类操作成功后需要刷新列表，并将页码设置为首页；

「useList」提供了「onRefresh」操作接口，其接收一个「boolean」类型的参数（默认值为「true」），若为「true」则「将页码设置为首页并发起请求」，否则「只发起请求」，示例代码如下：  
```
import React from "react";
import { Button } from "@arco-design/web-react";

const useList = buildUseList({
  getData: fetchData,
});

const Cmpt: React.FC = (props) => {
  // 初始化列表相关状态
  const [, { onRefresh }] = useList(props);

  return (
    <>
      {/** ... 略 */}

      <Button onClick={() => onRefresh(true)}>刷新</Button>
    </>
  );
};

export default Cmpt;
```
**移动端列表请求**  
移动端列表相比桌面端列表而言多了个场景，就是列表数据请求为「加载更多」而非「下一页」。  
这里的「列表数据」在「加载更多」的动作下请求参数的页码是递增的，列表数据是不断拼接的，而非直接替换原有的列表数据。  
「useList」在「build」阶段可配置「platform」参数（默认为Desktop）为「Mobile」来支持这种场景，示例代码如下：  
```
import React from "react";
import { Button } from "@arco-design/web-react";

const useList = buildUseList({
  // 配置应用场景为移动端
  platform: EListPlatform.Mobile,
  // 数据请求逻辑
  getData: fetchData,
});

const Cmpt: React.FC = (props) => {
  // 初始化列表相关状态
  const [{ list, pageNo }, { onLoadMore }] = useList(props);

  return (
    <>
      {/** ... 略 */}

      {list}

      {/** 执行「加载更多」函数 */}
      <Button onClick={() => onLoadMore()}>当前页码：{pageNo}，加载更多</Button>
    </>
  );
};

export default Cmpt;
```

原文: 
[React通用解决方案——组件数据请求](https://juejin.cn/post/7066722095526314021?utm_source=gold_browser_extension)