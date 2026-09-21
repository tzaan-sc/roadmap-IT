# 04 - Điện Toán Đám Mây: AWS, GCP & Azure (Cloud Computing)

> **Mục tiêu bài học:** Giải mã bản chất của Điện toán đám mây (Cloud Computing), phân biệt rõ các mô hình dịch vụ (IaaS, PaaS, SaaS, Serverless), làm chủ bản đồ các dịch vụ cốt lõi của AWS (EC2, S3, RDS, IAM, VPC), và hiểu được thế mạnh đặc trưng của ba "ông lớn" công nghệ: Amazon Web Services (AWS), Google Cloud Platform (GCP) và Microsoft Azure.

---

## 1. Điện Toán Đám Mây Là Gì? Tại Sao Thế Giới Chuyển Dịch Lên Cloud?

Trước đây, khi muốn vận hành một trang web cho công ty:
- Bạn phải ước tính lượng người dùng, mua máy chủ vật lý đắt đỏ (hàng trăm triệu đồng) và thuê chỗ đặt máy chủ trong phòng lạnh Data Center.
- Nếu lượng người dùng tăng đột biến: Mua thêm server mất **1 đến 2 tháng** để nhập khẩu và lắp ráp phần cứng!
- Nếu dự án thất bại: Hàng đống server vật lý đắt tiền bị bỏ xó!

**Điện toán đám mây (Cloud Computing)** là việc thuê tài nguyên máy tính (CPU, RAM, Dung lượng lưu trữ, Mạng) từ các nhà cung cấp khổng lồ (Amazon, Google, Microsoft) qua đường truyền Internet:

```text
┌────────────────────────────────────────────────────────────────────────┐
│                        3 ƯU ĐIỂM VÀNG CỦA CLOUD                        │
└───────┬──────────────────────────┬───────────────────────────┬─────────┘
        │                          │                           │
        ▼                          ▼                           ▼
1. PAY-AS-YOU-GO            2. ELASTICITY               3. GLOBAL REACH
(Dùng bao nhiêu trả bấy     (Co giãn linh hoạt tức thì) (Phạm vi phủ sóng toàn cầu)
 nhiêu, tính theo từng      Tăng từ 1 server lên 100    Chỉ cần 1 click chuột là
 giây hoặc megabyte dùng)   server trong vòng 2 phút!   deploy web sang Mỹ, Nhật, Âu!
```

---

## 2. Bốn Mô Hình Dịch Vụ Đám Mây (Cloud Service Models)

Sự khác biệt nằm ở việc **bạn chịu trách nhiệm quản lý bao nhiêu phần** và **nhà cung cấp Cloud quản lý hộ bạn bao nhiêu phần**:

```text
       ON-PREMISES              IaaS                     PaaS                 SERVERLESS / SaaS
    (Tự xây tại cty)       (Ví dụ: AWS EC2)        (Ví dụ: Render)        (Ví dụ: AWS Lambda)
┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐ ┌──────────────────────┐
│ Mã nguồn ứng dụng    │ │ Mã nguồn ứng dụng    │ │ Mã nguồn ứng dụng    │ │ Chỉ viết hàm logic!  │
│ Dữ liệu              │ │ Dữ liệu              │ │ Dữ liệu              │ ├──────────────────────┤
│ Runtime (Node/Python)│ │ Runtime              │ ├──────────────────────┤ │                      │
│ Hệ điều hành (OS)    │ │ Hệ điều hành (OS)    │ │                      │ │  NHÀ CUNG CẤP CLOUD  │
│ Ảo hóa (Virtualiz)   │ ├──────────────────────┤ │  NHÀ CUNG CẤP CLOUD  │ │     LO TẤT CẢ TỪ     │
│ Máy chủ vật lý       │ │                      │ │     LO TOÀN BỘ       │ │     A ĐẾN Z!         │
│ Ổ cứng lưu trữ       │ │  NHÀ CUNG CẤP CLOUD  │ │    HẠ TẦNG & OS!     │ │                      │
│ Mạng & Điện năng     │ │     LO PHẦN CỨNG!    │ │                      │ │                      │
└──────────────────────┘ └──────────────────────┘ └──────────────────────┘ └──────────────────────┘
 (Bạn tự lo từ A đến Z)   (Bạn thuê máy ảo thô)   (Bạn chỉ cần đưa code)  (Không còn khái niệm server)
```

1. **IaaS (Infrastructure as a Service):** Nhà cung cấp cho bạn thuê hạ tầng thô (CPU, RAM, ổ đĩa). Bạn toàn quyền cài hệ điều hành Ubuntu/Windows, tự cài đặt phần mềm và tự vá lỗi bảo mật (Ví dụ: **AWS EC2, Google Compute Engine**).
2. **PaaS (Platform as a Service):** Nhà cung cấp lo toàn bộ hệ điều hành, runtime và tự động cập nhật. Bạn chỉ cần liên kết kho GitHub hoặc quăng file Docker lên là web tự chạy (Ví dụ: **Render, Vercel, Heroku, AWS Elastic Beanstalk**).
3. **Serverless (FaaS - Function as a Service):** Bạn không cần biết server là gì. Bạn chỉ viết một đoạn hàm (Function), upload lên cloud. Khi có HTTP request gửi tới, cloud sẽ tự động đánh thức một container siêu nhỏ để chạy hàm đó trong vài trăm mili-giây rồi tự tắt. Chi phí chỉ tính trên số mili-giây hàm thực sự chạy (Ví dụ: **AWS Lambda, Google Cloud Functions**).

---

## 3. Bản Đồ Dịch Vụ Cốt Lõi Của AWS (Amazon Web Services)

AWS là nền tảng đám mây số 1 thế giới với hơn 200 dịch vụ. Dưới đây là **5 nhóm dịch vụ cốt lõi bắt buộc mọi kỹ sư phần mềm đều phải nắm vững**:

```text
                                  KIẾN TRÚC ỨNG DỤNG TRÊN AWS
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ AWS CLOUD (VIRTUAL PRIVATE CLOUD - VPC: Mạng riêng ảo cô lập bảo mật)                 │
│                                                                                        │
│   ┌──────────────────────────────────┐            ┌────────────────────────────────┐   │
│   │ PUBLIC SUBNET (Kết nối Internet) │            │ PRIVATE SUBNET (Cô lập mạng)   │   │
│   │                                  │            │                                │   │
│   │  ┌────────────────────────────┐  │            │  ┌──────────────────────────┐  │   │
│   │  │ AMAZON EC2                 │  │            │  │ AMAZON RDS               │  │   │
│   │  │ (Chạy Docker Web & API)    │ ─┼────────────┼─►│ (PostgreSQL Database)    │  │   │
│   │  └────────────────────────────┘  │ Mạng nội bộ│  │ - Tự động backup         │  │   │
│   │                │                 │            │  │ - Không ai từ ngoài vào  │  │   │
│   └────────────────┼─────────────────┘            │    được!                    │  │   │
│                    ▼                              │  └──────────────────────────┘  │   │
│   ┌──────────────────────────────────┐            └────────────────────────────────┘   │
│   │ AMAZON S3                        │                                                 │
│   │ (Lưu trữ ảnh, video, tài liệu)   │    ┌────────────────────────────────────────┐   │
│   │ - Dung lượng vô hạn              │    │ AWS IAM (Identity & Access Management) │   │
│   │ - Độ bền 99.999999999% (11 số 9) │    │ Quản lý phân quyền bảo mật & phân vai  │   │
│   └──────────────────────────────────┘    └────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

### 3.1. Amazon EC2 (Elastic Compute Cloud)
- Là máy chủ ảo trên đám mây. Bạn có thể chọn loại máy chuyên tối ưu CPU (Compute-optimized: c6g), tối ưu RAM (Memory-optimized: r6g), hoặc đa dụng (General-purpose: t3/t4g).
- Hỗ trợ tính năng **Auto-scaling Group**: Tự động sinh thêm máy ảo mới khi lượng truy cập tăng và tự tắt bớt khi đêm xuống để tiết kiệm tiền.

### 3.2. Amazon S3 (Simple Storage Service)
- Không dùng để lưu trữ hệ điều hành, mà dùng để **lưu trữ các tệp tin đối tượng tĩnh (Object Storage)**: ảnh đại diện của user, video, file PDF hóa đơn, bản sao lưu database.
- Độ bền cam kết đạt **99.999999999%** (xác suất mất 1 file là 1 lần trong 10,000 năm!).
- Chi phí siêu rẻ, tích hợp sẵn CDN **Amazon CloudFront** để truyền tải ảnh siêu tốc trên toàn cầu.

### 3.3. Amazon RDS (Relational Database Service)
- Dịch vụ CSDL quan hệ (PostgreSQL / MySQL) được quản lý tự động hoàn toàn:
  - Tự động sao lưu dữ liệu hàng ngày (Automated Backups).
  - Khôi phục dữ liệu về bất kỳ thời điểm nào trong quá khứ (Point-in-Time Recovery).
  - Tính năng **Multi-AZ (High Availability)**: Tự động nhân bản sang 2 trung tâm dữ liệu độc lập; nếu trung tâm dữ liệu 1 bị mất điện, hệ thống tự chuyển sang trung tâm dữ liệu 2 trong 60 giây mà ứng dụng không bị gián đoạn.

### 3.4. AWS IAM (Identity and Access Management)
- Hệ thống kiểm soát an ninh truy cập:
  - **Users:** Tài khoản cá nhân cho từng kỹ sư trong công ty.
  - **Groups:** Nhóm quyền (nhóm Developers, nhóm Admins).
  - **Roles (Vai trò):** Gán quyền tạm thời cho một dịch vụ máy tính. Ví dụ: Cấp quyền (IAM Role) cho máy chủ EC2 được phép upload file lên S3 mà **hoàn toàn không cần phải lưu hardcode access key hay secret key trong file code**!

---

## 4. So Sánh "Tam Đại Gia" Đám Mây: AWS vs GCP vs Azure

| Tiêu chí | Amazon Web Services (AWS) | Microsoft Azure | Google Cloud Platform (GCP) |
| :--- | :--- | :--- | :--- |
| **Thị phần toàn cầu** | **Số 1 (~31%)** | Số 2 (~25%) | Số 3 (~11%) |
| **Thế mạnh vượt trội** | Dịch vụ đa dạng và trưởng thành nhất, hệ sinh thái tài liệu khổng lồ | Tích hợp hoàn hảo với hệ sinh thái Microsoft (Windows Server, Active Directory, Office 365, .NET) | **Đỉnh cao về Trí tuệ nhân tạo (AI/ML)**, Phân tích dữ liệu lớn (BigQuery), và quản lý container (GKE) |
| **Đối tượng khách hàng** | Mọi loại hình doanh nghiệp, Startup, Công ty công nghệ | Các tập đoàn truyền thống, Doanh nghiệp tài chính đã dùng hệ sinh thái Microsoft | Các công ty công nghệ chuyên sâu về Data, AI, Machine Learning |
| **Khuyến nghị học tập** | **Nên học đầu tiên** vì thị trường tuyển dụng lớn nhất | Rất giá trị nếu định hướng làm cho doanh nghiệp lớn | Rất giá trị nếu định hướng làm kỹ sư Data / AI |

---

## 5. Tóm Tắt & Ghi Nhớ Nhanh

1. **Cloud Computing** mang lại khả năng co giãn linh hoạt (Elasticity) và thanh toán theo mức độ sử dụng thực tế (Pay-as-you-go).
2. Phân biệt rõ: **IaaS** (thuê máy ảo tự cài - EC2), **PaaS** (chỉ đưa code - Render/Vercel), và **Serverless** (chỉ viết hàm - AWS Lambda).
3. Nắm chắc 5 dịch vụ nền móng của AWS: **EC2** (máy chủ), **S3** (lưu trữ tệp), **RDS** (CSDL tự quản), **VPC** (mạng riêng ảo) và **IAM** (phân quyền bảo mật).
4. Luôn đặt Database trong **Private Subnet** và dùng **IAM Roles** thay vì hardcode thông tin đăng nhập trong mã nguồn.
5. **AWS** là lựa chọn số 1 để bắt đầu học Cloud nhờ thị phần tuyển dụng áp đảo trên toàn cầu.
