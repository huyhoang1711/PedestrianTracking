## I. Giới thiệu

**Object Tracking (Theo dõi đối tượng)** là một bài toán quan trọng trong lĩnh vực Thị giác Máy tính (Computer Vision), với mục tiêu theo dõi và xác định vị trí của các đối tượng qua các khung hình liên tiếp trong một video.

Dự án này tập trung vào việc xây dựng một hệ thống **Pedestrian Tracking (Theo dõi người đi bộ)**. Hệ thống có khả năng phát hiện và theo dõi những người đi bộ trong video.

Phương pháp chính được sử dụng là sự kết hợp giữa mô hình phát hiện đối tượng **YOLOv8** và thuật toán theo dõi **DeepSORT**.

*   **YOLOv8 (You Only Look Once version 8)**: Là một mô hình phát hiện đối tượng hiệu quả và nhanh chóng, có khả năng xác định vị trí của đối tượng trong ảnh hoặc video với độ chính xác cao.
*   **DeepSORT (Simple Online and Realtime Tracking with a Deep Association Metric)**: Là một thuật toán theo dõi đối tượng tiên tiến, sử dụng deep learning để tính toán sự tương đồng giữa các đối tượng trong các khung hình liên tiếp.

## II. Các bước thực hiện

### 1. Chuẩn bị dữ liệu

Dự án sử dụng bộ dữ liệu **MOT17 (Multiple Object Tracking 17)**. Dữ liệu được xử lý để phù hợp với định dạng mà YOLO yêu cầu.

*   **Tải dữ liệu:**
    ```bash
    !wget https://motchallenge.net/data/MOT17.zip
    !unzip -qq MOT17.zip
    ```
*   **Xử lý dữ liệu:**
    *   Các thư mục con không cần thiết sẽ được loại bỏ, chỉ giữ lại phiên bản FRCNN của dữ liệu.
    *   Chuyển đổi định dạng của bounding box và tạo các file label tương ứng cho mỗi ảnh.
    *   Tái cấu trúc lại thư mục, di chuyển tất cả các file ảnh và file label vào các thư mục `images` và `labels` tương ứng trong `train` và `test`.
    *   Tạo file `mot17_data.yml` để cấu hình cho quá trình huấn luyện.

### 2. Module Detector: Huấn luyện YOLOv8

*   **Cài đặt thư viện:**
    ```bash
    !pip install ultralytics -q
    ```
*   **Huấn luyện mô hình:**
    *   Sử dụng mô hình `yolov8s.pt` đã được huấn luyện trước.
    *   Tiến hành huấn luyện trên bộ dữ liệu MOT17 đã được xử lý.
    *   Kết quả huấn luyện (file weights `best.pt` hoặc `last.pt`) sẽ được lưu lại để sử dụng cho bước sau.

### 3. Module Tracker: Cài đặt DeepSORT

*   **Cài đặt thư viện và source code:**
    ```bash
    !pip install scikit-learn numpy opencv-python tensorflow spacy -q
    !pip install gdown==4.6.0 -q
    !git clone https://github.com/wjnwjn59/deep_sort.git
    ```
*   **Tải checkpoint của mô hình CNN:**
    *   Tải file checkpoint cần thiết để thuật toán DeepSORT có thể trích xuất đặc trưng của đối tượng.
    ```bash
    !gdown --no-check-certificate --folder https://drive.google.com/open?id=18fKzfqnqhqW3s9zwsCbnVJ5XF2JFeqMp
    ```

### 4. Thực hiện Tracking

*   **Tích hợp:** Xây dựng các lớp (class) `YOLOv8` và `DeepSORT` để dễ dàng sử dụng.
*   **Chạy trên video:**
    *   Đọc video đầu vào theo từng khung hình (frame).
    *   Với mỗi frame:
        1.  Sử dụng **YOLOv8** đã huấn luyện để phát hiện người đi bộ (lấy ra bounding boxes, confidence scores và class IDs).
        2.  Cung cấp kết quả phát hiện cho **DeepSORT** để thực hiện việc theo dõi và gán ID cho từng người.
    *   **Vẽ kết quả:** Vẽ bounding box và ID của từng đối tượng lên frame.
    *   **Lưu video:** Tổng hợp các frame đã xử lý để tạo thành video kết quả.
