# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

一个房间已经有待确认的寄养订单，房间管理页和创建订单弹窗仍显示 `0 / 容量`，直到正式入住才突然加一。请修复占用统计，使待确认、已确认和进行中的有效预订都被计入；现有测试保持不变，不得减少覆盖，页面展示要与下单判断一致。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/pet-foster-task-14
- 仓库地址：https://github.com/zhanglei10281852-gif/pet-foster-task-14.git
- parent SHA：4397daff756c2f8084448d1525fa561cc1d9d45b

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/pet-foster-task-14.git bug-repro
cd bug-repro
git checkout --detach 4397daff756c2f8084448d1525fa561cc1d9d45b
go test ./internal/pet -run ^TestAnnotationPendingBookingCountsTowardOccupancy$ -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationPendingBookingCountsTowardOccupancy$ -count=1
--- FAIL: TestAnnotationPendingBookingCountsTowardOccupancy (0.23s)
    annotation_pet_behavior_test.go:252: occupancy=0
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	0.231s
FAIL

```

stderr：

```text
(empty)
```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/pet -run ^TestAnnotationPendingBookingCountsTowardOccupancy$ -count=1
--- FAIL: TestAnnotationPendingBookingCountsTowardOccupancy (1.07s)
    annotation_pet_behavior_test.go:252: occupancy=0
FAIL
FAIL	github.com/zhanglei10281852-gif/pet-foster-go/internal/pet	1.327s
FAIL

```

stderr：

```text
(empty)
```

## 通过条件

房间占用数必须计入待确认、已确认和进行中的有效订单，并排除不占位状态，使房间列表、创建订单选项和容量判断采用一致结果。TestAnnotationPendingBookingCountsTowardOccupancy 应从红转绿，internal/pet 与全量测试无回归；现有测试不可改动，也不能删减待确认订单覆盖或绕开实际统计查询。
