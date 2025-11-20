# Hướng dẫn Cơ chế Routing trong OneWeb-develop-erp

Tài liệu này giải thích cách thức hoạt động của routing trong dự án, dựa trên `vue-router`.

## 1. Tổng quan
Dự án sử dụng **Vue Router** để quản lý điều hướng. Cấu trúc routing được thiết kế theo dạng **module hóa**, nghĩa là mỗi module chức năng (như QLTN, CSKH,...) sẽ có file cấu hình router riêng, sau đó được gom lại tại file cấu hình chính.

## 2. Cấu trúc File
- **[src/router/index.js](file:///d:/DTC_CODE/OneWeb-develop-erp/src/router/index.js)**: File cấu hình chính. Nơi khởi tạo router, import các route từ các module con và định nghĩa các global guards (kiểm tra quyền truy cập).
- **`src/modules/[Tên_Module]/router.js`**: File định nghĩa route cho từng module cụ thể (ví dụ: [src/modules/QLTN/router.js](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLTN/router.js)).

## 3. Cơ chế hoạt động

### 3.1. Main Router ([src/router/index.js](file:///d:/DTC_CODE/OneWeb-develop-erp/src/router/index.js))
File này làm nhiệm vụ:
1.  **Import Vue và VueRouter**.
2.  **Import các route con**: Import mảng các route từ các module khác nhau.
    ```javascript
    import QLTN from '@/modules/QLTN/router.js'
    import CSKH from '@/modules/CSKH/router.js'
    // ...
    ```
3.  **Khởi tạo Router**: Gom tất cả các route con vào mảng `routes`.
    ```javascript
    const router = new Router({
      routes: [
        { path: '', component: MainLayout, children: [...] }, // Route mặc định
        ...QLTN, // Spread operator để gộp các route của module QLTN
        ...CSKH,
        // ...
      ]
    })
    ```
4.  **Navigation Guards (`beforeEach`)**: Kiểm tra quyền trước khi chuyển trang.
    - Log hành trình (`Routing from... to...`).
    - Kiểm tra `menu.checkExists`: Route có tồn tại trong menu không.
    - Kiểm tra `requiresAuth`: Có cần đăng nhập không.
    - Kiểm tra Token hết hạn: Nếu hết hạn sẽ redirect về trang Login.

### 3.2. Module Router (Ví dụ: [src/modules/QLTN/router.js](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLTN/router.js))
Mỗi module định nghĩa một danh sách các route liên quan đến nó.
- **Lazy Loading**: Sử dụng `import()` để tải component chỉ khi cần thiết, giúp giảm tải ban đầu.
    ```javascript
    const TraCuuPhieuDaGiao = () => import('./TraCuuPhieuDaGiao')
    ```
- **Cấu trúc Route**: Thường có một route cha dùng chung Layout (ví dụ [MainLayout](file:///d:/DTC_CODE/OneWeb-develop-erp/src/modules/QLTN/router.js#3-6)) và các route con.
    ```javascript
    export default [
      {
        path: '/qltn',
        component: MainLayout,
        meta: { middleware: [auth] }, // Middleware kiểm tra quyền
        children: [
          {
            path: 'TraCuuPhieuDaGiao', // Đường dẫn con: /qltn/TraCuuPhieuDaGiao
            name: 'TraCuuPhieuDaGiao',
            component: TraCuuPhieuDaGiao
          },
          // ... các route khác
        ]
      }
    ]
    ```

## 4. Cách thêm một Route mới

Giả sử bạn muốn thêm một trang chức năng mới tên là `MyNewPage` vào module `QLTN`.

### Bước 1: Tạo Component
Tạo file `MyNewPage.vue` trong thư mục `src/modules/QLTN`.

### Bước 2: Khai báo trong `src/modules/QLTN/router.js`
Mở file `src/modules/QLTN/router.js`:
1.  **Import component** (khuyên dùng lazy loading):
    ```javascript
    const MyNewPage = () => import('./MyNewPage.vue')
    ```
2.  **Thêm vào mảng `children`** của route cha `/qltn`:
    ```javascript
    children: [
      // ... các route cũ
      {
        path: 'my-new-page', // Đường dẫn sẽ là /qltn/my-new-page
        name: 'MyNewPage',
        component: MyNewPage,
        meta: {
          title: 'Trang chức năng mới' // Tiêu đề trang (tùy chọn)
        }
      }
    ]
    ```

### Bước 3: Kiểm tra
Chạy `npm run dev` và truy cập `http://localhost:8081/qltn/my-new-page` để kiểm tra.

## 5. Lưu ý quan trọng
- **Gitignore**: File `src/router/index.js` đang bị gitignore. Nếu bạn sửa file này, hãy đảm bảo bạn hiểu lý do tại sao nó bị ignore (có thể là file được sinh tự động hoặc cấu hình riêng cho local). Tuy nhiên, file `src/router/index.js.bak` đang lưu cấu trúc mẫu.
- **Middleware**: Các route thường được bảo vệ bởi middleware `auth`. Đảm bảo bạn đã đăng nhập để truy cập các route này.
