# Hướng dẫn Custom Event trong Vue Component

Tài liệu này hướng dẫn chi tiết về cơ chế tạo và sử dụng custom event trong dự án, kèm theo các ví dụ thực tế từ source code.

## 1. Định nghĩa và Kích hoạt Event (`$emit`)

Để component con gửi thông báo ra component cha, ta sử dụng phương thức `$emit(eventName, payload)`.

### Ví dụ 1: Event cơ bản (Không tham số)
Trong file [src/modules/QLDA/components/FrmSelectExt.vue](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue), component kích hoạt event [clear](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue#197-200) khi người dùng xóa dữ liệu:

```javascript
// Định nghĩa trong methods của component con
clearValue: function() {
  this.$emit("clear"); // Kích hoạt event tên là 'clear'
}
```

**Sử dụng ở component cha:**
```html
<FrmSelectExt @clear="handleClear" />
```

### Ví dụ 2: Event có kèm dữ liệu (Payload)
Cũng trong [FrmSelectExt.vue](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue), khi giá trị thay đổi, event [change](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue#167-170) được emit kèm theo tham số `args`:

```javascript
// Định nghĩa trong methods
change: function(args) {
  this.$emit("change", args); // args là dữ liệu được gửi kèm
}
```

**Sử dụng ở component cha:**
```html
<FrmSelectExt @change="handleChange" />

// Trong methods của cha
methods: {
  handleChange(value) {
    console.log("Giá trị mới:", value);
  }
}
```

## 2. Custom `v-model`

Mặc định, `v-model` trong Vue tương đương với việc nhận prop [value](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue#135-138) và lắng nghe event [input](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue#102-105). Tuy nhiên, ta có thể tùy chỉnh điều này bằng option `model`.

### Ví dụ thực tế: [FrmSelectExt.vue](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue)
Component này muốn `v-model` hoạt động với prop [value](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue#135-138) nhưng lắng nghe event [change](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue#167-170) thay vì [input](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLDA/components/FrmSelectExt.vue#102-105).

**Định nghĩa trong component con:**
```javascript
export default {
  name: "FrmSelectExt",
  // Tùy chỉnh v-model
  model: {
    event: "change", // Tên event để cập nhật v-model
    prop: "value",   // Tên prop liên kết với v-model
  },
  props: {
    value: null, // Khai báo prop value
    // ...
  },
  methods: {
    change: function(args) {
      // Khi emit 'change', v-model ở cha sẽ tự động cập nhật
      this.$emit("change", args);
    }
  }
}
```

**Sử dụng ở component cha:**
```html
<!-- Giá trị selectedValue sẽ tự động cập nhật khi event 'change' được emit -->
<FrmSelectExt v-model="selectedValue" />
```

## 3. Modifier `.sync` (Two-way binding cho Prop)

Trong Vue 2, để cập nhật trực tiếp một prop từ con lên cha (ngoài `v-model`), ta dùng modifier `.sync`. Component con cần emit event theo format `update:propName`.

### Ví dụ thực tế: [DanhSachKhachhang.vue](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/search/subscriber/SearchMobileSolutions/components/DanhSachKhachhang.vue)
Component này sử dụng `b-table` và muốn đồng bộ trạng thái sắp xếp (`sortBy`, `sortDesc`) giữa cha và con.

**Sử dụng trong template:**
```html
<b-table
  :sort-by.sync="sortBy"
  :sort-desc.sync="sortDesc"
  ...
>
</b-table>
```

**Cơ chế hoạt động:**
1.  Cha truyền `sortBy` vào `b-table`.
2.  Khi người dùng click vào header để sắp xếp, `b-table` sẽ emit event `update:sortBy` với giá trị mới.
3.  Vue tự động cập nhật biến `sortBy` ở cha với giá trị mới đó.

Tương đương với viết tường minh:
```html
<b-table
  :sort-by="sortBy"
  @update:sortBy="newValue => sortBy = newValue"
  ...
>
</b-table>
```

## 4. Tổng kết

| Cơ chế | Cú pháp kích hoạt (Con) | Cú pháp lắng nghe (Cha) | Mục đích |
| :--- | :--- | :--- | :--- |
| **Event thường** | `this.$emit('my-event', data)` | `@my-event="handler"` | Thông báo hành động, gửi dữ liệu tùy ý. |
| **v-model** | `this.$emit('input', data)` (mặc định) | `v-model="data"` | Binding dữ liệu 2 chiều chính (thường là value). |
| **.sync** | `this.$emit('update:propName', data)` | `:prop-name.sync="data"` | Binding dữ liệu 2 chiều cho các prop khác. |
