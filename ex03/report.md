# BÁO CÁO BÀI 3: TỐI ƯU TỐC ĐỘ CI BẰNG GRADLE DEPENDENCY CACHING

## 1. Mục tiêu

Cấu hình Gradle Dependency Caching trong GitHub Actions nhằm tái sử dụng các thư viện phụ thuộc giữa những lần chạy CI, giảm thời gian tải thư viện từ Internet và tối ưu thời gian kiểm thử, đóng gói ứng dụng Spring Boot.

## 2. Nội dung thực hiện

Chỉnh sửa tệp `.github/workflows/ci.yml` để bổ sung cấu hình cache Gradle trong bước thiết lập Java JDK 17:

```yaml
- name: Set up Java JDK 17
  uses: actions/setup-java@v5
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: 'gradle'
```

Workflow thực hiện các công việc sau:

1. **Checkout:** Tải mã nguồn từ repository.
2. **Set up Java JDK 17:** Thiết lập môi trường Java và cấu hình Gradle Dependency Caching.
3. **Cấp quyền Gradle Wrapper:** Sử dụng `chmod +x gradlew` để cho phép thực thi Gradle Wrapper.
4. **Chạy Unit Test:** Thực thi `./gradlew test` để kiểm tra mã nguồn.
5. **Đóng gói ứng dụng:** Thực thi `./gradlew bootJar` để tạo tệp JAR của ứng dụng Spring Boot.
6. **Lưu Artifact:** Sử dụng `actions/upload-artifact@v4` để lưu tệp JAR với tên `app-jar`.

## 3. Kết quả so sánh thời gian thực thi

Thực hiện push mã nguồn hai lần liên tiếp và theo dõi kết quả trong GitHub Actions.

| Tiêu chí | Lần push thứ nhất | Lần push thứ hai |
|---|---|---|
| Gradle Dependency Cache | Có thể chưa tồn tại cache phù hợp | Có thể khôi phục cache đã lưu |
| Set up Java JDK 17 | ... giây | ... giây |
| `./gradlew test` | ... giây | ... giây |
| `./gradlew bootJar` | ... giây | ... giây |
| Tổng thời gian Job | ... phút ... giây | ... phút ... giây |

*Lưu ý: Điền thời gian thực tế dựa trên log của GitHub Actions. Thời gian có thể thay đổi tùy thuộc vào trạng thái cache, quá trình tải thư viện và thời gian chạy kiểm thử.*

## 4. Kiểm tra Gradle Dependency Cache

Sau lần push thứ nhất, GitHub Actions có thể tải các thư viện phụ thuộc và tạo cache để sử dụng cho những lần chạy sau.

Sau lần push thứ hai, kiểm tra log của bước `Set up Java JDK 17`. Nếu cache phù hợp được khôi phục, log sẽ hiển thị thông báo tương tự:

```text
Cache restored successfully from key: gradle-...
```

Ngoài ra, kiểm tra thời gian chạy của các bước `./gradlew test` và `./gradlew bootJar` để đánh giá mức độ cải thiện.

## 5. Đánh giá kết quả

Việc bổ sung `cache: 'gradle'` cho phép GitHub Actions lưu và khôi phục cache phụ thuộc Gradle dựa trên cấu hình dự án.

Khi cache được sử dụng lại thành công, Gradle có thể giảm thời gian tải các thư viện đã có sẵn. Điều này giúp tối ưu quy trình CI, đặc biệt đối với các dự án Spring Boot có nhiều thư viện phụ thuộc.

Tuy nhiên, cache không đảm bảo tất cả các bước đều chạy nhanh hơn trong mọi lần thực thi. Thời gian kiểm thử và đóng gói còn phụ thuộc vào mã nguồn, số lượng bài kiểm thử và trạng thái cache.

## 6. Kết luận

Đã bổ sung cấu hình Gradle Dependency Caching vào workflow CI sử dụng Java JDK 17.

Workflow tự động thực hiện kiểm thử đơn vị, đóng gói ứng dụng Spring Boot thành tệp JAR và lưu kết quả dưới dạng Artifact. Việc sử dụng cache giúp giảm thao tác tải lại các thư viện phụ thuộc khi cache phù hợp được khôi phục thành công.

