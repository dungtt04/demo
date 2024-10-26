Để triển khai hệ thống quản lý học viên với Laravel chi tiết nhất, hãy thực hiện theo từng bước cụ thể dưới đây, bao gồm thiết lập cơ sở dữ liệu, API, tạo giao diện, và phân quyền cho các nhóm người dùng. 

---

## 1. Thiết lập môi trường Laravel và cấu hình cơ sở dữ liệu

### Bước 1: Cài đặt Laravel và các thư viện cần thiết
1. **Cài đặt dự án Laravel**:
   ```bash
   composer create-project --prefer-dist laravel/laravel qlhv
   ```

2. **Cấu hình kết nối cơ sở dữ liệu** trong file `.env`:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=qlhv
   DB_USERNAME=root
   DB_PASSWORD=your_password
   ```

3. **Cài đặt thư viện Excel** để nhập/xuất dữ liệu:
   ```bash
   composer require maatwebsite/excel
   ```

4. **Cài đặt thư viện DomPDF** để xuất file PDF:
   ```bash
   composer require barryvdh/laravel-dompdf
   ```

5. **Khởi tạo hệ thống xác thực người dùng**:
   ```bash
   php artisan make:auth
   ```

---

## 2. Tạo các bảng cơ sở dữ liệu bằng Laravel Migrations

Dựa vào cấu trúc file Excel, chúng ta sẽ tạo các bảng sau: `hoc_vien`, `diem_tcpt`, `diem_tn`, và `diem_xtn`.

### Tạo bảng `hoc_vien` (Thông tin học viên)
1. Tạo migration cho bảng `hoc_vien`:
   ```bash
   php artisan make:migration create_hoc_vien_table --create=hoc_vien
   ```

2. Cấu hình migration cho bảng `hoc_vien`:
   ```php
   Schema::create('hoc_vien', function (Blueprint $table) {
       $table->id();
       $table->string('MHV')->unique();
       $table->string('ho_va_ten');
       $table->date('ngay_sinh');
       $table->string('noi_sinh')->nullable();
       $table->string('truong_dang_hoc')->nullable();
       $table->timestamps();
   });
   ```

### Tạo bảng `diem_tcpt` (Điểm thực chiến phòng thi)
1. Tạo migration:
   ```bash
   php artisan make:migration create_diem_tcpt_table --create=diem_tcpt
   ```

2. Cấu hình migration:
   ```php
   Schema::create('diem_tcpt', function (Blueprint $table) {
       $table->id();
       $table->foreignId('hoc_vien_id')->constrained('hoc_vien')->onDelete('cascade');
       $table->float('diem_thuc_hanh_1')->nullable();
       $table->float('diem_thuc_hanh_2')->nullable();
       $table->float('diem_thuc_hanh_3')->nullable();
       $table->float('diem_thuc_hanh_4')->nullable();
       $table->timestamps();
   });
   ```

### Tạo bảng `diem_tn` (Điểm tuyển sinh)
1. Tạo migration:
   ```bash
   php artisan make:migration create_diem_tn_table --create=diem_tn
   ```

2. Cấu hình migration:
   ```php
   Schema::create('diem_tn', function (Blueprint $table) {
       $table->id();
       $table->foreignId('hoc_vien_id')->constrained('hoc_vien')->onDelete('cascade');
       $table->float('diem_toan')->nullable();
       $table->float('diem_ngu_van')->nullable();
       $table->float('diem_lich_su')->nullable();
       $table->float('diem_dia_ly')->nullable();
       $table->float('diem_trung_binh_12')->nullable();
       $table->float('diem_xet_tot_nghiep')->nullable();
       $table->timestamps();
   });
   ```

### Tạo bảng `diem_xtn` (Điểm xét tốt nghiệp)
1. Tạo migration:
   ```bash
   php artisan make:migration create_diem_xtn_table --create=diem_xtn
   ```

2. Cấu hình migration:
   ```php
   Schema::create('diem_xtn', function (Blueprint $table) {
       $table->id();
       $table->foreignId('hoc_vien_id')->constrained('hoc_vien')->onDelete('cascade');
       $table->string('SBD')->nullable();
       $table->float('diem_xtn')->nullable();
       $table->timestamps();
   });
   ```

3. Chạy tất cả các migrations để tạo các bảng:
   ```bash
   php artisan migrate
   ```

---

## 3. Tạo Models và thiết lập mối quan hệ giữa các bảng

### 1. Tạo models cho từng bảng:
```bash
php artisan make:model HocVien
php artisan make:model DiemTcpt
php artisan make:model DiemTn
php artisan make:model DiemXtn
```

### 2. Thiết lập quan hệ giữa các bảng
Trong model `HocVien`:
```php
class HocVien extends Model
{
    public function diemTcpt()
    {
        return $this->hasOne(DiemTcpt::class);
    }

    public function diemTn()
    {
        return $this->hasOne(DiemTn::class);
    }

    public function diemXtn()
    {
        return $this->hasOne(DiemXtn::class);
    }
}
```

---

## 4. Xây dựng API để nhập dữ liệu từ file Excel

### 1. Tạo Controller quản lý học viên
```bash
php artisan make:controller HocVienController
```

### 2. Thiết lập API nhập dữ liệu từ file Excel
**Cài đặt thư viện Excel để import từ file Excel vào cơ sở dữ liệu.**

**Tạo file Import**:
```bash
php artisan make:import HocVienImport --model=HocVien
```

Cấu hình `HocVienImport.php`:
```php
namespace App\Imports;

use App\Models\HocVien;
use Maatwebsite\Excel\Concerns\ToModel;

class HocVienImport implements ToModel
{
    public function model(array $row)
    {
        return new HocVien([
            'MHV' => $row[0],
            'ho_va_ten' => $row[1],
            'ngay_sinh' => \PhpOffice\PhpSpreadsheet\Shared\Date::excelToDateTimeObject($row[2]),
            'noi_sinh' => $row[3],
            'truong_dang_hoc' => $row[4],
        ]);
    }
}
```

**Tạo API** trong `HocVienController` để nhập file Excel:
```php
use App\Imports\HocVienImport;
use Maatwebsite\Excel\Facades\Excel;

public function import(Request $request)
{
    $file = $request->file('file');
    Excel::import(new HocVienImport, $file);
    return redirect()->back()->with('success', 'Dữ liệu nhập thành công');
}
```

**Route nhập Excel** trong `web.php`:
```php
Route::post('/import-hoc-vien', [HocVienController::class, 'import']);
```

---

## 5. Giao diện người dùng với Blade Templates

### 1. Giao diện cho Admin
- **Trang nhập điểm**: Form upload file Excel cho học viên.

  **Form upload Excel** trong view `admin/import_excel.blade.php`:
  ```html
  <form action="{{ url('/import-hoc-vien') }}" method="POST" enctype="multipart/form-data">
      @csrf
      <input type="file" name="file" required>
      <button type="submit">Upload</button>
  </form>
  ```

- **Trang quản lý điểm**: Hiển thị danh sách học viên và cho phép chỉnh sửa từng điểm.

### 2. Giao diện Client
- **Trang xem điểm**: Hiển thị điểm của từng học viên từ các bảng `diem_tcpt`, `diem_tn`, `diem_xtn` qua các hàm trong `HocVienController`.

---

## 6. Chức năng xuất PDF

1. **Cấu hình DomPDF trong `HocVienController`**:
   ```php
   use Barryvdh\DomPDF\Facade\Pdf;

   public function exportPdf($id)
   {
       $hocVien = HocVien::with(['diemTcpt', 'diemTn', 'diemXtn'])->find($id);
       $pdf = PDF::loadView('hoc_vien.export_pdf', compact('hocVien'));
       return $pdf->download('BangDiemHocVien.pdf');
   }
   ```

2. **Route xuất PDF**:
   ```php
   Route::get('/export-pdf/{id}', [HocVienController::class, 'exportPdf']);
   ```

---

## 7. Phân quyền người dùng

### 1. Cài đặt Middleware phân quyền
- Trong `Kernel.php`, thêm middleware để kiểm tra quyền hạn của admin và client:
   ```php
   'admin' => \App\Http\Middleware

\IsAdmin::class,
   ```

### 2. Route phân quyền
```php
Route::middleware(['auth', 'admin'])->group(function () {
   Route::get('/admin', [AdminController::class, 'index']);
});

Route::middleware(['auth', 'client'])->group(function () {
   Route::get('/client', [ClientController::class, 'index']);
});
```

---

Với các bước này, hệ thống quản lý học viên sẽ có thể hoạt động với đầy đủ tính năng yêu cầu. Nếu cần mở rộng hoặc chi tiết trong từng phần cụ thể, hãy cho tôi biết nhé!
