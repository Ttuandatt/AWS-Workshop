+++

title = "Blog 4"

weight = 4

chapter = false

pre = "<b> 3.4. </b>"

+++

# Triển khai liên tục theo GitOps với ArgoCD và EKS bằng ngôn ngữ tự nhiên

**Tác giả:** Jagdish Komakula, Aditya Ambati, Anand Krishna Varanasi  

**Ngày:** 17/07/2025

**Danh mục:**  

Amazon Elastic Kubernetes Service (EKS), Amazon Q, Amazon Q Developer, Developer Tools, Technical How-to

---

## Giới thiệu

ArgoCD là một công cụ GitOps hàng đầu giúp các nhóm quản lý việc triển khai **Kubernetes** theo mô hình khai báo, trong đó **Git đóng vai trò là nguồn chân lý duy nhất (single source of truth)**. ArgoCD cung cấp nhiều tính năng mạnh mẽ như:

- Đồng bộ hóa tự động

- Khả năng khôi phục (rollback)

- Phát hiện sai lệch cấu hình

- Chiến lược triển khai nâng cao

- Tích hợp RBAC

- Hỗ trợ đa cụm (multi-cluster)

Nhờ đó, ArgoCD trở thành lựa chọn phổ biến cho việc triển khai ứng dụng trên Kubernetes. Tuy nhiên, khi tổ chức mở rộng quy mô, nhiều thách thức vận hành bắt đầu xuất hiện.

---

## Điểm khó khăn khi sử dụng ArgoCD theo cách truyền thống

- **Rào cản kỹ thuật cao**  

  Giao diện UI và CLI của ArgoCD yêu cầu người dùng hiểu rõ YAML, kiến trúc Kubernetes và các mối quan hệ giữa tài nguyên. Điều này làm hạn chế khả năng tiếp cận GitOps với các bên liên quan không chuyên về kỹ thuật và tăng phụ thuộc vào đội ngũ DevOps.

- **Quản lý đa cụm phức tạp**  

  Các mô hình như hub-spoke, per-cluster hoặc grouped làm tăng độ phức tạp trong vận hành, khi phải quản lý nhiều instance ArgoCD và duy trì cấu hình nhất quán.

- **Thiếu tích hợp tác vụ tiền/hậu triển khai**  

  ArgoCD không hỗ trợ sẵn các bước như quét image hoặc kiểm thử tải, buộc các nhóm phải sử dụng công cụ ngoài hoặc script tùy chỉnh, gây phân mảnh quy trình.

- **Chuyển đổi môi trường kém linh hoạt**  

  Việc promotion ứng dụng từ Dev → Test → Prod thường phải làm thủ công hoặc qua script, làm chậm quá trình phát hành.

- **Quản lý RBAC và khả năng hiển thị khó khăn**  

  Trong môi trường đa cụm, việc quản lý quyền truy cập và giám sát có thể dẫn đến rủi ro bảo mật tiềm ẩn.

---

## Cách ArgoCD MCP Server và Amazon Q CLI giải quyết vấn đề

Việc tích hợp **ArgoCD MCP Server** với **Amazon Q CLI** mang lại trải nghiệm GitOps hoàn toàn mới thông qua **ngôn ngữ tự nhiên**.

Những lợi ích chính:

- Cho phép người dùng quản lý ứng dụng ArgoCD bằng câu lệnh hội thoại thay vì CLI hoặc YAML.

- Dân chủ hóa GitOps cho các vai trò như QA, Product Manager hay Support Engineer.

- Đơn giản hóa quản lý đa cụm và đa môi trường.

- MCP Server đảm nhiệm xác thực, quản lý session và xử lý lỗi.

- Amazon Q cung cấp gợi ý theo ngữ cảnh và phản hồi chi tiết.

- Hỗ trợ tự động hóa và gỡ lỗi bằng AI, đóng vai trò như một **DevOps ảo**.

Ví dụ câu lệnh:

- "Những ứng dụng nào đang out-of-sync ở production?"

- "Đồng bộ ứng dụng api-service."

---

## So sánh ArgoCD truyền thống và ArgoCD MCP + Amazon Q CLI

| Tiêu chí | ArgoCD truyền thống | MCP Server + Amazon Q CLI |

|--------|-------------------|--------------------------|

| Giao diện | UI/CLI kỹ thuật | Ngôn ngữ tự nhiên |

| Người dùng không kỹ thuật | Hạn chế | Dễ tiếp cận |

| Quản lý đa cụm | Phức tạp | Được trừu tượng hóa |

| Tiền/Hậu triển khai | Dùng công cụ ngoài | Dễ gọi qua AI |

| Promotion môi trường | Thủ công | Hội thoại |

| Khắc phục sự cố | Phụ thuộc kỹ thuật | Có AI hỗ trợ |

| Tự động hóa | Script | AI/Agent |

---

## Các thao tác hỗ trợ bằng ngôn ngữ tự nhiên

Thông qua tích hợp Amazon Q CLI, bạn có thể:

- Quản lý ứng dụng ArgoCD

- Đồng bộ và theo dõi trạng thái

- Trực quan hóa cây tài nguyên

- Giám sát sức khỏe tài nguyên

- Theo dõi sự kiện

- Truy xuất log

- Thực hiện hành động trên tài nguyên Kubernetes

---

## Thiết lập môi trường

### Điều kiện tiên quyết

- Tài khoản AWS

- AWS CLI ≥ 2.13.0

- Node.js ≥ 18.0.0

- npm ≥ 9.0.0

- Amazon Q CLI ≥ 1.0.0  

  ```bash

  npm install -g @aws/amazon-q-cli

Cụm EKS (≥ 1.27) với ArgoCD ≥ 2.8

Kết nối với EKS

```bash
aws eks update-kubeconfig --name <cluster_name> --region <region> --role-arn <iam_role_arn>
```

```bash
kubectl get pods -n argocd
```

```bash
kubectl port-forward svc/blueprints-addon-argocd-server -n argocd 8080:443
```
Tích hợp Amazon Q CLI với ArgoCD MCP

```bash
{

  "mcpServers": {

    "argocd-mcp-stdio": {

      "type": "stdio",

      "command": "npx",

      "args": ["argocd-mcp@latest", "stdio"],

      "env": {

        "ARGOCD_BASE_URL": "<ARGOCD_BASE_URL>",

        "ARGOCD_API_TOKEN": "<ARGOCD_API_TOKEN>",

        "NODE_TLS_REJECT_UNAUTHORIZED": "0"

      }

    }

  }

}
```

Tổng kết

Việc tích hợp Amazon Q CLI với ArgoCD thông qua MCP Server đánh dấu một bước tiến quan trọng trong quản lý Kubernetes. Giải pháp này chuyển đổi các thao tác phức tạp thành tương tác hội thoại tự nhiên, giúp:

Giảm gánh nặng kỹ thuật

Tăng tốc triển khai

Mở rộng GitOps cho toàn bộ tổ chức

Thay vì ghi nhớ lệnh và xử lý YAML, các nhóm giờ đây có thể quản lý hạ tầng cloud thông qua đối thoại, giúp hành trình cloud-native trở nên dễ tiếp cận và hiệu quả hơn.
