Phân tích lý do thất bại:
Prompt "Code này có chạy được không?" thất bại vì 5 lý do chính:

- Câu hỏi đóng, phạm vi quá hẹp: AI hiểu đây là yêu cầu kiểm tra cú pháp,
không phải kiểm tra logic nghiệp vụ. Kết quả là AI xác nhận code hợp lệ về mặt cú pháp và dừng lại ở đó.

- Không có Role: Không yêu cầu AI đóng vai QA hay Tester,
nên AI mặc định phản hồi như một người đọc code thông thường, không chủ động tìm điểm yếu.

- Không có Context: Không cung cấp thông tin về các trường hợp đầu vào có thể xảy ra
như danh sách rỗng hoặc null, nên AI không có cơ sở để xét đến các tình huống biên.

- Không có Constraint: Không ràng buộc AI phải kiểm tra edge case, exception,
hay bất kỳ tiêu chí chất lượng nào cụ thể.

- Không có Format: Không yêu cầu đầu ra cụ thể nên AI trả lời tùy ý,
thường chỉ là một câu xác nhận ngắn, không có phân tích hay giải pháp đi kèm.

Prompt mới:

Hãy đóng vai một Chuyên gia Kiểm thử phần mềm (QA/Tester) Java có kinh nghiệm,
chuyên phát hiện lỗi logic và các trường hợp biên nguy hiểm.

Đây là đoạn mã nguồn cần kiểm tra:

---
public class AverageCalculator {
public static double calculateAverage(List<Integer> numbers) {
    int sum = 0;
    for (int num : numbers) {
        sum += num;
    }
    return (double) sum / numbers.size();
    }
}
---

Bối cảnh: Hàm này được dùng trong hệ thống thực tế, nhận đầu vào từ người dùng.
Đầu vào hoàn toàn có thể là danh sách rỗng hoặc giá trị null.

Hãy thực hiện 3 nhiệm vụ sau:

Nhiệm vụ 1: Liệt kê tất cả các trường hợp biên (edge case) có thể làm sập chương trình,
giải thích loại lỗi tương ứng (Exception) và nguyên nhân gốc rễ.

Nhiệm vụ 2: Viết lại hàm calculateAverage() đã được sửa lỗi hoàn chỉnh,
xử lý toàn bộ các trường hợp biên đã liệt kê, có chú thích tiếng Việt ở mỗi bước xử lý.

Nhiệm vụ 3: Viết JUnit 5 test case để kiểm thử từng trường hợp biên đó.

Ràng buộc:
- Sử dụng Java 17 và Spring Boot 3.x
- Không được thay đổi tên class hoặc tên hàm gốc
- Toàn bộ chú thích và giải thích bằng tiếng Việt
- Trình bày mỗi nhiệm vụ trong một khối code markdown riêng biệt

Phản hồi từ AI:

Các trường hợp biên nguy hiểm:

- numbers == null → ném ra NullPointerException ngay tại vòng lặp for-each vì không thể duyệt qua null
- numbers.isEmpty() → ném ra ArithmeticException: / by zero tại dòng sum / numbers.size() vì chia cho 0
- Danh sách chứa toàn số âm hoặc số rất lớn → không gây crash nhưng kết quả có thể tràn số (integer overflow) nếu tổng vượt quá Integer.MAX_VALUE

Mã nguồn đã sửa lỗi:

import java.util.List;

public class AverageCalculator {

    public static double calculateAverage(List<Integer> numbers) {

        // Kiểm tra đầu vào null - ném ngoại lệ rõ ràng thay vì để NullPointerException âm thầm
        if (numbers == null) {
            throw new IllegalArgumentException("Danh sách đầu vào không được là null.");
        }

        // Kiểm tra danh sách rỗng - tránh lỗi chia cho 0
        if (numbers.isEmpty()) {
            throw new IllegalArgumentException("Danh sách đầu vào không được rỗng.");
        }

        // Dùng long thay vì int để tránh tràn số khi tổng quá lớn
        long sum = 0;
        for (int num : numbers) {
            sum += num;
        }

        // Chia lấy giá trị trung bình, ép kiểu về double để giữ phần thập phân
        return (double) sum / numbers.size();
    }
}


JUnit 5 Test Case:

javaimport org.junit.jupiter.api.Test;
import java.util.Arrays;
import java.util.Collections;

import static org.junit.jupiter.api.Assertions.*;

class AverageCalculatorTest {

    // Kiểm tra trường hợp danh sách null - phải ném IllegalArgumentException
    @Test
    void testNullInput_shouldThrowIllegalArgumentException() {
        assertThrows(IllegalArgumentException.class, () -> {
            AverageCalculator.calculateAverage(null);
        });
    }

    // Kiểm tra trường hợp danh sách rỗng - phải ném IllegalArgumentException
    @Test
    void testEmptyList_shouldThrowIllegalArgumentException() {
        assertThrows(IllegalArgumentException.class, () -> {
            AverageCalculator.calculateAverage(Collections.emptyList());
        });
    }

    // Kiểm tra trường hợp danh sách có một phần tử - kết quả phải bằng chính phần tử đó
    @Test
    void testSingleElement_shouldReturnThatElement() {
        double result = AverageCalculator.calculateAverage(Arrays.asList(10));
        assertEquals(10.0, result);
    }

    // Kiểm tra trường hợp bình thường với nhiều số nguyên dương
    @Test
    void testNormalInput_shouldReturnCorrectAverage() {
        double result = AverageCalculator.calculateAverage(Arrays.asList(2, 4, 6, 8));
        assertEquals(5.0, result);
    }

    // Kiểm tra trường hợp danh sách chứa số âm
    @Test
    void testNegativeNumbers_shouldReturnCorrectAverage() {
        double result = AverageCalculator.calculateAverage(Arrays.asList(-4, -2, 0, 2, 4));
        assertEquals(0.0, result);
    }

    // Kiểm tra trường hợp tổng lớn có thể gây tràn số nếu dùng int
    @Test
    void testLargeValues_shouldNotOverflow() {
        double result = AverageCalculator.calculateAverage(
            Arrays.asList(Integer.MAX_VALUE, Integer.MAX_VALUE)
        );
        assertEquals((double) Integer.MAX_VALUE, result);
    }
}