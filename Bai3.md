# Bài 3: Thực hành Refactor Clean Code - Refinement Process

## 1. Phân tích các điểm vi phạm Clean Code của đoạn mã ban đầu

Đoạn mã gốc có nhiều vấn đề sau:

- Cấu trúc lồng nhau quá sâu
  - Có nhiều lớp `if` lồng nhau, làm khó đọc và khó bảo trì.
- Tên biến vô nghĩa
  - `list`, `a`, `branch`, `activeOnly` là tên chưa rõ ràng theo Clean Code.
- Logic business bị rải rác
  - Điều kiện lọc tài khoản nằm dày đặc trong nhiều nhánh, khó kiểm thử.
- Thiếu guard clauses
  - Nên trả về sớm khi đầu vào không hợp lệ thay vì tiếp tục vào các nhánh lồng.
- Không có logging
  - Khi gặp dữ liệu bất thường hoặc quá trình tính toán, không có log để trace.
- Không tận dụng Stream API
  - Code dài dòng nhưng vẫn chưa biểu diễn rõ ý định xử lý dữ liệu.

---

## 2. Chuỗi Prompt cải tiến 3 vòng

### Vòng 1: Robustness & Clean Code

```text
Hãy refactor đoạn Java sau theo nguyên tắc Clean Code và Robustness.

Yêu cầu:
1. Loại bỏ cấu trúc if-else lồng nhau sâu bằng Guard Clauses hoặc Return Early.
2. Đổi tên biến rõ nghĩa và chuẩn hóa theo convention Java.
3. Đảm bảo method không bị lặp logic.
4. Giữ nguyên mục đích nghiệp vụ: tính tổng số dư tài khoản của một chi nhánh.
5. Không cần thêm thư viện ngoài nếu không cần thiết.

Đầu ra chỉ cần mã nguồn Java tối ưu cho class LedgerBalanceCalculator.
```

### Vòng 2: Maintainability & OOP

```text
Tiếp tục refactor đoạn code trên theo hướng Maintainability và OOP.

Yêu cầu:
1. Tối ưu thuật toán bằng Java 17 Stream API để làm cho logic ngắn gọn và dễ đọc hơn.
2. Tách rõ logic lọc và tính tổng.
3. Nếu class này hoạt động trong môi trường Spring Boot, hãy thêm chú thích @Transactional(readOnly = true).
4. Không làm thay đổi ý nghĩa nghiệp vụ.
5. Chỉ xuất mã nguồn Java đã refactor hoàn chỉnh.
```

### Vòng 3: Logging & Context Tuning

```text
Hãy tiếp tục cải tiến class LedgerBalanceCalculator theo hướng production-ready.

Yêu cầu:
1. Thêm Lombok @Slf4j để ghi log.
2. Log thông tin số lượng tài khoản được duyệt qua và tổng tiền tính được ở mức log.info.
3. Đảm bảo log không làm giảm hiệu năng quá mức.
4. Giữ logic tính tổng đúng và dễ đọc.
5. Đầu ra là mã nguồn Java hoàn chỉnh, có import cần thiết và có chú thích ngắn nếu cần.
```

---

## 3. Minh chứng chạy thực tế

### Log phản hồi AI (mô phỏng)

```text
Vòng 3 - Mã nguồn hoàn chỉnh:
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Objects;

@Slf4j
public class LedgerBalanceCalculator {

    @Transactional(readOnly = true)
    public double calculateTotalBalance(List<Account> accounts, String branch, boolean activeOnly) {
        if (accounts == null || accounts.isEmpty()) {
            log.info("No accounts provided for branch {}", branch);
            return 0.0;
        }

        if (branch == null || branch.isBlank()) {
            log.warn("Branch is null or blank, returning zero balance");
            return 0.0;
        }

        long processedAccountCount = accounts.stream()
                .filter(Objects::nonNull)
                .filter(account -> branch.equals(account.getBranch()))
                .filter(account -> !activeOnly || "ACTIVE".equals(account.getStatus()))
                .filter(account -> account.getBalance() > 0)
                .count();

        double totalBalance = accounts.stream()
                .filter(Objects::nonNull)
                .filter(account -> branch.equals(account.getBranch()))
                .filter(account -> !activeOnly || "ACTIVE".equals(account.getStatus()))
                .filter(account -> account.getBalance() > 0)
                .mapToDouble(Account::getBalance)
                .sum();

        log.info("Processed {} accounts for branch {}. Total balance = {}", processedAccountCount, branch, totalBalance);
        return totalBalance;
    }
}
```

Ghi chú:
- Code đã loại bỏ cấu trúc lồng nhau sâu.
- Logic lọc tài khoản được tách rõ và dễ đọc hơn.
- Đã thêm logging và tối ưu rõ ràng bằng Stream API.
```
