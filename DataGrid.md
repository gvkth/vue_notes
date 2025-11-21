# Hướng dẫn Component DataGrid

## 1. Tổng quan
Component `DataGrid` (`src/components/Controls/DataGrid`) là một wrapper component được xây dựng dựa trên thư viện **Syncfusion EJ2 Grid** (`@syncfusion/ej2-vue-grids`).

Nó cung cấp các tính năng:
- Hiển thị dữ liệu dạng bảng.
- Phân trang (Client-side & Server-side).
- Lọc, Sắp xếp, Grouping.
- Chọn dòng (Single/Multiple).
- Các cột chức năng (Command Column).
- Export Excel.

## 2. Cách sử dụng cơ bản

```html
<template>
  <DataGrid
    :columns="columns"
    :dataSource="dataSources"
    :showFilter="true"
    :allowPaging="true"
    :enablePagingServer="false"
    :totalRecords="dataSources.length"
    @selectedRowChanged="onSelectedRowChanged"
  />
</template>

<script>
export default {
  data() {
    return {
      columns: [
        { fieldName: 'ma_nv', headerText: 'Mã NV', width: '100px' },
        { fieldName: 'ten_nv', headerText: 'Tên Nhân viên', allowFiltering: true },
        { fieldName: 'ngay_sinh', headerText: 'Ngày sinh', textAlign: 'right' }
      ],
      dataSources: [
        { ma_nv: 'NV001', ten_nv: 'Nguyễn Văn A', ngay_sinh: '01/01/1990' },
        { ma_nv: 'NV002', ten_nv: 'Trần Thị B', ngay_sinh: '05/05/1995' }
      ]
    }
  },
  methods: {
    onSelectedRowChanged(row) {
      console.log('Dòng được chọn:', row);
    }
  }
}
</script>
```

## 3. Cấu hình chi tiết (Props)

### Dữ liệu & Cột
| Prop | Type | Default | Mô tả |
| :--- | :--- | :--- | :--- |
| `dataSource` | Array | `[]` | Mảng dữ liệu hiển thị trên lưới. |
| `columns` | Array | `[]` | Cấu hình các cột (Xem chi tiết object Column bên dưới). |
| `dataKeyField` | String | `''` | Tên trường khóa chính (ID). Dùng khi lấy danh sách ID đã chọn. |

### Phân trang
| Prop | Type | Default | Mô tả |
| :--- | :--- | :--- | :--- |
| `allowPaging` | Boolean | `false` | Bật/tắt phân trang. |
| `enablePagingServer` | Boolean | `true` | `true`: Phân trang server (gọi API khi đổi trang). `false`: Phân trang tại client. |
| `pageIndex` | Number | `0` | Trang hiện tại (0-based). |
| `pageSize` | Number | `10` | Số bản ghi trên 1 trang. |
| `totalRecords` | Number | `0` | Tổng số bản ghi (bắt buộc nếu `enablePagingServer=true`). |

### Tính năng
| Prop | Type | Default | Mô tả |
| :--- | :--- | :--- | :--- |
| `showFilter` | Boolean | `true` | Hiển thị dòng filter ở header. |
| `allowSorting` | Boolean | `true` | Cho phép click header để sắp xếp. |
| `showColumnCheckbox` | Boolean | `false` | Hiển thị cột checkbox để chọn nhiều dòng. |
| `selectionSettings` | Object | `{}` | Cấu hình chọn dòng (VD: `{ type: 'Multiple' }`). |
| `allowResizing` | Boolean | `true` | Cho phép kéo thả thay đổi độ rộng cột. |
| `allowExcelExport` | Boolean | `false` | Cho phép xuất Excel. |
| `panelDataHeight` | String | `'auto'` | Chiều cao của bảng (VD: `'300px'`). |

### Cấu trúc Object `columns`
Mỗi phần tử trong mảng `columns` có thể có các thuộc tính:
- `fieldName`: Tên trường dữ liệu (key trong object data).
- `headerText`: Tiêu đề cột.
- `width`: Độ rộng (VD: `'100px'`, `'50%'`).
- `textAlign`: Căn lề (`'left'`, `'right'`, `'center'`).
- `allowFiltering`: Cho phép lọc cột này hay không (`true`/`false`).
- `visible`: Ẩn/hiện cột.
- `isPrimaryKey`: Đánh dấu là khóa chính.
- `template`: Hàm hoặc component template tùy chỉnh cho ô dữ liệu.
- `commands`: Mảng các nút bấm chức năng (nếu là cột Command).

## 4. Các Event thường dùng

| Event Name | Tham số | Mô tả |
| :--- | :--- | :--- |
| `rowClicked` | `(rowIndex, rowData)` | Khi click vào một dòng. |
| `rowDoubleClicked` | `(rowIndex, rowData)` | Khi double click vào một dòng. |
| `selectedRowChanged` | `(rowData)` | Khi dòng được chọn thay đổi (Single selection). |
| `selectedItemsChanged` | `(selectedIds)` | Khi danh sách chọn thay đổi (Multiple selection). Trả về mảng ID nếu có `dataKeyField`, ngược lại trả về mảng object. |
| `pageChanged` | `{ pageIndex, pageSize }` | Khi người dùng chuyển trang hoặc đổi số lượng bản ghi/trang. |
| `commandClicked` | `(cmdName, rowData)` | Khi click vào nút chức năng trong cột Command. |
| `queryCellInfo` | `args` | Hook để can thiệp vào việc render từng ô (VD: tô màu nền dựa trên giá trị). |

## 5. Ví dụ nâng cao: Cột chức năng (Command Column)

```javascript
// Trong data()
columns: [
  // ... các cột khác
  {
    fieldName: '',
    headerText: 'Thao tác',
    width: '100px',
    commands: [
      { name: 'EDIT', title: 'Sửa', class: 'btn-primary', text: 'Sửa' },
      { name: 'DELETE', title: 'Xóa', class: 'btn-danger', text: 'Xóa' }
    ]
  }
]

// Trong template
<DataGrid
  ...
  @commandClicked="onCommandClicked"
/>

// Trong methods
onCommandClicked(cmdName, rowData) {
  if (cmdName === 'EDIT') {
    this.editItem(rowData);
  } else if (cmdName === 'DELETE') {
    this.deleteItem(rowData);
  }
}
```

## 6. Lưu ý
- Component này sử dụng `provide/inject` để cung cấp các module của Syncfusion Grid (`Sort`, `Filter`, `Page`, v.v.).
- Nếu `enablePagingServer` là `true`, bạn cần bắt sự kiện `pageChanged` để gọi lại API lấy dữ liệu trang mới và cập nhật lại `dataSource`.
- `pageIndex` trong component này bắt đầu từ 0, nhưng API của dự án thường bắt đầu từ 1, hãy chú ý chuyển đổi.
