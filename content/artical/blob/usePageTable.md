---
desc: 
title: usePageTable的简单封装
createTime: 2023-06-17 21:17:21
updateTime: 2023-06-29 22:10:47
---

## usePageTable的简单封装

​		在一个函数中封装多个api集合(基本的增删改查)的hook函数，包含表单的加载状态，modal弹出框的状态，表单数据源的封装，以及分页切换的数据重新查询。

###### 2. 实现方法

(1) 定义串口参数接口

```typescript [code]
/** 接收的查询接口参数,默认有pageSize和pageNum */
interface PageFormOptions {
  pageSize?: number;
  pageNum?: number;
  [options: string]: any;
}
```

(2) 定义增删改查接口参数类型

```typescript [code]
// 此处接收的泛型T用于定义表单数据源的类型
type PromiseResult<T = any> = Promise<Service.FailedResult | Service.SuccessResult<T>>;

interface PageTableParams<T> {
  GetApiFn: (args: any) => PromiseResult<T>;
  AddApiFn: (args: any) => PromiseResult;
  EditApiFn: (args: any) => PromiseResult;
  DeleteApiFn: (args: any) => PromiseResult;
  options?: PageFormOptions;
}
```

(3) 表单状态加载和模态框的变量定义，通过ref实现(此处调用了一个useBoolean函数--简单的赋值操作封装)

```typescript [code]
const { setFalse: stopLoading, setTrue: startLoading, bool: loading } = useBoolean();
const { setFalse: modalHide, setTrue: modalShow, bool: modalVisile } = useBoolean();
```

(4) 表单数据源存储定义

```typescript [code]
const tableOrigin = ref<T>();
```

(5) 获取表单数据列表操作

```typescript [code]
async function getListFn() {
    startLoading();
    const { data, error } = await params.GetApiFn(apiFnParams);
    stopLoading();
    if (!error) {
      tableOrigin.value = data;
    }
  }
```

(6) 修改、删除、添加等(代码会放到最后，耦合较多，不作展示)

```typescript [code]
async function addListFn(args: unknown) {
    if (!params.AddApiFn) return;
    const { error } = await params.AddApiFn(args);
    if (!error) {
      modalHide();
      await getListFn();
      window.$message?.success('添加成功');
    }
  }
```

(7) 结尾导出页面中要使用的变量和函数

```typescript [code]
return {
    getListFn,
    apiFnParams,
    tableOrigin,
    loading,
    modalVisile,
    modalHide,
    modalShow,
    addListFn,
    editListFn,
    deleteListFn
  };
```

###### 3. 界面使用(demo)

```typescript [code]
const { loading, getListFn, tableOrigin, addListFn, editListFn, deleteListFn, modalHide, modalShow, modalVisile } =
  usePageTable<ApiFrp.FrpMap[]>({
    GetApiFn: fetchGetFrpMapList,
    AddApiFn: fetchAddFrpMapList,
    EditApiFn: fetchChangeFrpMapList,
    DeleteApiFn: fetchDeleteFrpMapList
  });

```

###### 3. 总结

​		此处封装的还是有些许简陋，后面还会对查询参数做一些调整，以及细节的补充。其实表单封装更多还是，针对某一些耦合度较高的增删改查表单，减少代码量和代码冗余而做的努力。如果说是比较复杂的表单，其实做统一封装的难度较大。可以做单独特殊的处理。

###### 4.代码(usePageTable.ts)

```typescript [usePageTable.ts]
import { reactive, ref } from 'vue';
import { useBoolean } from '@/hooks';

/** 接收的查询接口参数,默认有pageSize和pageNum */
interface PageFormOptions {
  pageSize?: number;
  pageNum?: number;
  [options: string]: any;
}

type PromiseResult<T = any> = Promise<Service.FailedResult | Service.SuccessResult<T>>;

interface PageTableParams<T> {
  GetApiFn: (args: any) => PromiseResult<T>;
  AddApiFn: (args: any) => PromiseResult;
  EditApiFn: (args: any) => PromiseResult;
  DeleteApiFn: (args: any) => PromiseResult;
  options?: PageFormOptions;
}

/**
 * table表格的hook封装, 默认pageNum=1， pageSize=10
 * @param params 参数项(PageTableParams)
 */
export function usePageTable<T>(params: PageTableParams<T>) {
  const { setFalse: stopLoading, setTrue: startLoading, bool: loading } = useBoolean();
  const { setFalse: modalHide, setTrue: modalShow, bool: modalVisile } = useBoolean();

  const apiFnParams = reactive({
    pageNum: 1,
    pageSize: 10
  });

  // 是否有参数
  if (params.options) {
    const { pageSize, pageNum } = params.options;
    if (pageNum && pageSize) {
      apiFnParams.pageNum = pageNum;
      apiFnParams.pageSize = pageSize;
    }
  }

  // 表单数据源
  const tableOrigin = ref<T>();

  async function getListFn() {
    startLoading();
    const { data, error } = await params.GetApiFn(apiFnParams);
    stopLoading();
    if (!error) {
      tableOrigin.value = data;
    }
  }

  async function addListFn(args: unknown) {
    if (!params.AddApiFn) return;
    const { error } = await params.AddApiFn(args);
    if (!error) {
      modalHide();
      await getListFn();
      window.$message?.success('添加成功');
    }
  }

  async function editListFn(args: unknown) {
    if (!params.EditApiFn) return;
    const { error } = await params.EditApiFn(args);
    if (!error) {
      modalHide();
      await getListFn();
      window.$message?.success('修改成功');
    }
  }

  async function deleteListFn(args: unknown) {
    if (!params.DeleteApiFn) return;
    const { error } = await params.DeleteApiFn(args);
    if (!error) {
      await getListFn();
      window.$message?.success('删除成功');
    }
  }

  return {
    getListFn,
    apiFnParams,
    tableOrigin,
    loading,
    modalVisile,
    modalHide,
    modalShow,
    addListFn,
    editListFn,
    deleteListFn
  };
}

```