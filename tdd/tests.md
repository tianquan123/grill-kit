# 好测试与坏测试

## 好测试

**集成风格**：通过真实接口测试，而不是 mock 内部零件。

```typescript
// 好：测试可观察的行为
test("user can checkout with valid cart", async () => {
  const cart = createCart();
  cart.add(product);
  const result = await checkout(cart, paymentMethod);
  expect(result.status).toBe("confirmed");
});
```

特征：

- 测的是用户/调用方关心的行为
- 只用公开 API
- 内部重构后依然存活
- 描述 **WHAT**（是什么），而不是 HOW（怎么做）
- 每个测试一个逻辑断言

## 坏测试

**实现细节测试**：与内部结构耦合。

```typescript
// 坏：测试实现细节
test("checkout calls paymentService.process", async () => {
  const mockPayment = jest.mock(paymentService);
  await checkout(cart, payment);
  expect(mockPayment.process).toHaveBeenCalledWith(cart.total);
});
```

危险信号：

- mock 内部协作者
- 测试私有方法
- 断言调用次数/调用顺序
- 行为没变、只是重构，测试就挂
- 测试名描述的是 HOW 而不是 WHAT
- 绕过接口、用外部手段验证

```typescript
// 坏：绕过接口去验证
test("createUser saves to database", async () => {
  await createUser({ name: "Alice" });
  const row = await db.query("SELECT * FROM users WHERE name = ?", ["Alice"]);
  expect(row).toBeDefined();
});

// 好：通过接口验证
test("createUser makes user retrievable", async () => {
  const user = await createUser({ name: "Alice" });
  const retrieved = await getUser(user.id);
  expect(retrieved.name).toBe("Alice");
});
```

**同义反复的测试**：期望值把实现重述了一遍，于是测试按构造就通过。

```typescript
// 坏：期望值用代码算它的同一方式重算
test("calculateTotal sums line items", () => {
  const items = [{ price: 10 }, { price: 5 }];
  const expected = items.reduce((sum, i) => sum + i.price, 0);
  expect(calculateTotal(items)).toBe(expected);
});

// 好：期望值是一个独立的、已知的字面量
test("calculateTotal sums line items", () => {
  expect(calculateTotal([{ price: 10 }, { price: 5 }])).toBe(15);
});
```
