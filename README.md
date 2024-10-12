## HƯỚNG DẪN VIEW BIẾN THỂ RA CHI TIẾT SẢN PHẨM

Để xử lý việc cập nhật giá khi người dùng chọn một biến thể, chúng ta sẽ chia quy trình thành các bước chi tiết hơn, đảm bảo mọi thứ đều hoạt động từ frontend đến backend. Dưới đây là quy trình chi tiết để triển khai chức năng này:

### 1. **Thiết lập dữ liệu ban đầu trong Blade Template:**
Khi trang chi tiết sản phẩm được tải lần đầu, chúng ta sẽ hiển thị danh sách các biến thể của sản phẩm với các nút bấm tương ứng và một giá mặc định (giá của sản phẩm hoặc giá của biến thể đầu tiên).

#### HTML trong `product-detail.blade.php`:
```blade
<h1>{{ $product->name }}</h1>
<p>{{ $product->description }}</p>

<!-- Giá sản phẩm hiển thị mặc định (giá của sản phẩm gốc hoặc biến thể đầu tiên) -->
<p>Giá: <span id="product-price">{{ $product->price }}</span></p>

<!-- Danh sách các biến thể -->
@foreach($product->variants as $variant)
    <button class="variant-btn" data-id="{{ $variant->id }}" data-price="{{ $variant->price }}">
        {{ $variant->variant_name }} ({{ $variant->sku }})
    </button>
@endforeach

<!-- Thông báo lỗi nếu có -->
<div id="error-message" style="color: red;"></div>
```

### 2. **Sử dụng JavaScript để thay đổi giá:**
Khi người dùng chọn một biến thể, chúng ta cần sử dụng JavaScript để cập nhật giá tương ứng với biến thể đã chọn. Đây là cách xử lý sự kiện click và cập nhật giá.

#### JavaScript để xử lý sự kiện:
```blade
<script>
    document.addEventListener('DOMContentLoaded', function() {
        const variantButtons = document.querySelectorAll('.variant-btn');
        const priceElement = document.getElementById('product-price');
        const errorMessage = document.getElementById('error-message');

        // Xử lý sự kiện click trên từng nút biến thể
        variantButtons.forEach(button => {
            button.addEventListener('click', function() {
                // Lấy thông tin biến thể được click
                const variantId = this.getAttribute('data-id');
                const variantPrice = this.getAttribute('data-price');

                // Kiểm tra nếu biến thể có giá
                if (variantPrice) {
                    priceElement.textContent = variantPrice; // Cập nhật giá sản phẩm
                    errorMessage.textContent = ''; // Xóa thông báo lỗi
                } else {
                    errorMessage.textContent = 'Không thể tải giá của biến thể này.';
                }

                // Thêm hiệu ứng active cho biến thể được chọn
                variantButtons.forEach(btn => btn.classList.remove('active')); // Xóa class active trên các nút khác
                this.classList.add('active'); // Thêm class active vào nút vừa được click
            });
        });

        // Chọn biến thể đầu tiên khi trang được tải lần đầu
        const firstVariant = document.querySelector('.variant-btn');
        if (firstVariant) {
            firstVariant.click(); // Mô phỏng click để hiển thị giá mặc định
        }
    });
</script>
```

#### **Giải thích chi tiết:**
- **`variantButtons`**: Tập hợp tất cả các nút biến thể.
- **`priceElement`**: Đoạn HTML hiển thị giá sản phẩm.
- **`errorMessage`**: Vùng hiển thị thông báo lỗi nếu có lỗi xảy ra.
- **Sự kiện click**: Khi người dùng click vào nút biến thể, giá trị của biến thể đó sẽ được cập nhật lên giao diện. Nếu biến thể có giá trị `price`, nó sẽ hiển thị giá tương ứng, nếu không sẽ thông báo lỗi.
- **Class "active"**: Sau khi người dùng chọn biến thể, nút biến thể đó sẽ được đánh dấu để người dùng dễ dàng nhận biết nó đã được chọn.

### 3. **Xử lý cập nhật giá động với AJAX:**
Trong trường hợp bạn muốn xử lý dữ liệu thông qua AJAX (ví dụ, lấy giá biến thể từ cơ sở dữ liệu thay vì lấy từ HTML), bạn có thể thay đổi như sau:

#### Route cho API:
Trong file `web.php`, bạn có thể thêm một route để xử lý việc lấy giá biến thể theo `variant_id`:
```php
Route::get('/product-variant/{id}', [ProductController::class, 'getVariantPrice']);
```

#### Controller lấy giá biến thể:
Trong `ProductController`, thêm một phương thức để xử lý yêu cầu AJAX, trả về thông tin biến thể, bao gồm giá:
```php
public function getVariantPrice($id)
{
    $variant = ProductVariant::findOrFail($id);
    
    return response()->json([
        'success' => true,
        'price' => $variant->price,
        'stock' => $variant->stock,
        'sku' => $variant->sku,
    ]);
}
```

#### JavaScript cho AJAX:
Thay vì lấy giá từ HTML, chúng ta sẽ sử dụng AJAX để gọi đến backend và lấy giá biến thể.

```blade
<script>
    document.addEventListener('DOMContentLoaded', function() {
        const variantButtons = document.querySelectorAll('.variant-btn');
        const priceElement = document.getElementById('product-price');
        const errorMessage = document.getElementById('error-message');

        variantButtons.forEach(button => {
            button.addEventListener('click', function() {
                const variantId = this.getAttribute('data-id');

                // Sử dụng AJAX để lấy thông tin biến thể từ backend
                fetch(`/product-variant/${variantId}`)
                    .then(response => response.json())
                    .then(data => {
                        if (data.success) {
                            priceElement.textContent = data.price; // Cập nhật giá từ backend
                            errorMessage.textContent = ''; // Xóa thông báo lỗi
                        } else {
                            errorMessage.textContent = 'Không thể tải giá của biến thể này.';
                        }
                    })
                    .catch(error => {
                        errorMessage.textContent = 'Lỗi kết nối. Vui lòng thử lại.';
                    });

                // Thêm hiệu ứng active cho biến thể được chọn
                variantButtons.forEach(btn => btn.classList.remove('active')); // Xóa class active trên các nút khác
                this.classList.add('active'); // Thêm class active vào nút vừa được click
            });
        });

        // Chọn biến thể đầu tiên khi trang được tải lần đầu
        const firstVariant = document.querySelector('.variant-btn');
        if (firstVariant) {
            firstVariant.click(); // Mô phỏng click để hiển thị giá mặc định
        }
    });
</script>
```

### 4. **Tóm tắt các bước:**
- **HTML**: Hiển thị danh sách các biến thể và giá của sản phẩm.
- **JavaScript**: Sử dụng sự kiện click để thay đổi giá trên giao diện ngay lập tức.
- **AJAX (Tùy chọn)**: Nếu cần, bạn có thể gọi AJAX để lấy giá biến thể từ backend khi người dùng click vào biến thể.
  
Điều này giúp giao diện trở nên linh hoạt và trải nghiệm người dùng mượt mà hơn khi tương tác với các biến thể sản phẩm. Nếu có thêm yêu cầu hay câu hỏi nào khác, hãy cho tôi biết!
