+++
title = "Blog 4"
weight = 4
chapter = false
pre = " <b> 3.4. </b>"
+++

# **Triển khai liên tục dựa trên phương pháp GitOps với ArgoCD và EKS bằng cách sử dụng ngôn ngữ tự nhiên**

_Jagdish Komakula, Aditya Ambati và Anand Krishna Varanasi_ | 17/07/2025 | [ Amazon Elastic Kubernetes Service](https://aws.amazon.com/blogs/devops/category/compute/amazon-kubernetes-service/), [Amazon Q](https://aws.amazon.com/blogs/devops/category/amazon-q/), [Amazon Q Developer](https://aws.amazon.com/blogs/devops/category/amazon-q/amazon-q-developer/), [Developer Tools](https://aws.amazon.com/blogs/devops/category/developer-tools/), [Technical How-to ](https://aws.amazon.com/blogs/devops/category/post-types/technical-how-to/) | [Permalink](https://aws.amazon.com/blogs/devops/gitops-continuous-delivery-with-argocd-and-eks-using-natural-language/)

## Giới thiệu

ArgoCD là một bộ công cụ GitOps hàng đầu giúp các nhóm quản lý việc triển khai Kubernets một cách khai báo, sử dụng Git là nguồn thông tin đáng tin cậy duy nhất. Bộ tính năng mạnh mẽ của nó bao gồm hệ thống đồng bộ tự động, hỗ trợ khôi phục, phát hiện sai lệch, chiến lược triển khai nâng cao, Tích hợp RBAC, và hỗ trợ đa cụm (multi-cluster), khiến nó trở thành giải pháp được ưa chuộng cho việc triển khai ứng dụng trên Kubernetes. Tuy nhiên, khi các tổ chức mở rộng quy mô, một vài điểm khó khăn và thử thách vận hành bắt đầu xuất hiện.

## Điểm khó khăn khi sử dụng ArgoCD theo phương pháp truyền thống

- Giao diện của ArgoCD và CLI được thiết kế cho người dùng có nền tảng công nghệ kỹ thuật chuyên sâu. Việc tích hợp tương tác với manifests YAML, hiểu mối quan hệ giữa các tài nguyên Kubernets, và việc khắc phục lỗi đồng bộ đòi hỏi kiến thức chuyên môn cao. Điều này hạn chế khả năng tiếp cập vào quy trình GitOps đối với các bên liên quan ít am hiểu công nghệ và làm tăng sự phụ thuộc vào các kỹ sư DevOps.

- Việc quản lý ArgoCD trên nhiều cụm (clusters) hoặc nhiều môi trường (environments) (sử dụng mô hình hub-spoke, per-cluster, hoặc grouped) tạo ra sự phức tạp đáng kể trong việc vận hành. Các nhóm phải xử lý nhiều instance ArgoCD, duy trì cấu hình nhất quán, và điều phối các triển khai, điều này có thể trở thành một nút thắt khi các quy mô dịch vụ ngày càng mở rộng.

- ArgoCD vượt trội trong việc đồng bộ và giám sát các tài nguyên Kubernetes nhưng thiếu cơ chế tích hợp sẵn cho các tác vụ tiền triển khai (ví dụ: quét hình ảnh) và hậu triển khai (ví dụ: việc kiểm tra tải). Điều này khiến các nhóm phải dựa vào các bộ công cụ bên ngoài hoặc các script tùy chỉnh, khiến quy trình triển khai bị phân mảnh và tăng thêm gánh nặng bảo trì.

- Việc chuyển ứng dụng giữa các môi trường (Dev -> Test -> Prod) không được tinh gọn một cách tự nhiên. Các nhóm phải tự mình điều phối thủ công hoặc viết script cho các quy trình chuyển đổi này, việc này làm chậm việc triển khai các bản vá lỗi khẩn cấp và làm phức tạp hóa quá trình phát hành.

- Khi các tổ chức áp dụng các chiến lược đa cụm (multi-cluster), việc quản lý quyền truy cập, RBAC và khả năng hiển thị tài nguyên của ArgoCD trên nhiều môi trường trở nên công kềnh, thường dẫn tới việc làm phân mảnh quy trình làm việc và tạo ra các lỗ hổng bảo mật tiềm ẩn.

## Cách ArgoCD MCP Server cùng với Amazon Q CLI giải quyết các vấn đề trên:

- Việc tích hợp máy chủ ArgoCD MCP với [Amazon Q CLI](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/command-line.html) về cơ bản đã thay đổi trải nghiệm người dùng bằng cách giới thiệu tương tác bằng ngôn ngữ tự nhiên cho các thao tác GitOps.

- Với MCP, người dùng có thể quản lý việc triển khai, giám sát trạng thái ứng dụng, và thực hiện thao tác đồng bộ hóa và khôi phục việc vận hành bằng cách sử dụng sử dụng ngôn ngữ giao tiếp thông thường thay vì các câu lệnh kỹ thuật (commands) hoặc YAML. Ví dụ, một người dùng có thể hỏi đơn giản là: "Những ứng dụng nào đang không đồng bộ ở môi trường production?" hay "Đồng bộ ứng dụng api-service," và hệ thống sẽ thực hiện ngầm các lệnh gọi API ArgoCD phù hợp.

- Điều này giúp dân chủ hóa quyền truy cập vào GitOps, cho phép các thành viên trong nhóm ít chuyên kỹ thuật (như QA, các người quản lý sản phẩm hoặc kỹ sư hỗ trợ) có thể tương tác an toàn với các quy trình triển khai.

- Giao diện ngôn ngữ tự nhiên giúp loại bỏ độ phức tạp của việc quản lý đa cụm (multi-cluster) và đa môi trường (multi-environment). Người dùng có thể truy vấn hoặc thực hiện hành động trên các tài nguyên giữa các cụm mà không cần ghi nhớ tên tài nguyên, namespace hoặc endpoint API.

- MCP Server xử lý việc xác thực, quản lý session và xử lý lỗi mạnh mẽ, giảm nhu cầu khắc phục sự cố thủ công và viết script tùy chỉnh.

- Việc tích hợp cung cấp các phản hồi chi tiết, xử lý các endpoint thông minh, và các thông báo lỗi toàn diện, giúp cho việc chẩn đoán và giải quyết vấn đề trở nên dễ dàng hơn. Việc kiểm tra kiểu tĩnh hoàn toàn và cấu hình theo môi trường còn nâng cao thêm độ tin cậy và khả năng bảo trì.

- Bằng cách tận dụng khả năng mở rộng của Amazon Q CLI, người dùng được tiếp cận với các tích hợp sẵn và gợi ý theo ngữ cảnh, từ đó đẩy nhanh quy trình phát triển và triển khai.

- MCP Server cho phép các trợ lý AI và mô hình ngôn ngữ tự động hóa các tác vụ định kỳ, đề xuất hành động và thậm chí gỡ lỗi (debug) các vấn đề, hoạt động như một kỹ sư DevOps ảo. Điều này có thể giảm đáng kể công sức thủ công và tăng tốc độ phản hồi sự cố.

## So sánh ArgoCD truyền thống với ArgoCD MCP Server và Amazon Q CLI

| Tính năng/Thách thức                         | ArgoCD truyền thống                                    | MPC Server + Amazon Q CLI                       |
| -------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------- |
| Giao diện người dùng                         | Giao diện kỹ thuật (UI/CLI), yêu cầu thao tác với YAML | Ngôn ngữ tự nhiên, tương tác hội thoại          |
| Khả năng truy cập cho người không phải kỹ sư | Hạn chế                                                | Mở rộng, dân chủ hóa                            |
| Quản lý đa cụm                               | Phức tạp, thủ công                                     | Đơn giản hóa, được trừu tượng hóa               |
| Tác vụ Tiền/Hậu Triển khai                   | Cần công cụ bên ngoài/script                           | Vẫn dùng công cụ bên ngoài, nhưng dễ gọi hơn    |
| Chuyển đổi ứng dụng                          | Thủ công hoặc dùng script                              | Ngôn ngữ tự nhiên, điều phối dễ dàng hơn        |
| Khắc phục sự cố                              | Mang tính kỹ thuật, dễ xảy ra lỗi                      | Được hướng dẫn, có hỗ trợ AI, phản hồi chi tiết |
| Tự động hóa                                  | Yêu cầu viết script                                    | Do AI/agent điều khiển, chủ động                |

Bạn có thể thực hiện các [thao tác](https://github.com/akuity/argocd-mcp?tab=readme-ov-file#available-tools) sau bằng ngôn ngữ tự nhiên nhờ tích hợp Amazon Q CLI với ArgoCD MCP server:

- **Quản lý ứng dụng**: Liệt kê, khởi tạo, cập nhật và xóa các ứng dụng ArgoCD.

- **Thao tác đồng bộ**: Kích hoạt đồng bộ và theo dõi trạng thái

- **Trực quan hóa cây tài nguyên**: Xem cấu trúc phân cấp của các tài nguyên do ứng dụng quản lý

- **Giám sát trạng thái sức khỏe**: Kiểm tra trạng thái sức khỏe của các ứng dụng và các tài nguyên của chúng.

- **Theo dõi sự kiện**: Xem các sự kiện liên quan đến ứng dụng và tài nguyên.

- **Truy cập log**: Truy xuất log từ các workload của ứng dụng.

- **Thực hiện hành động trên tài nguyên**: Gọi các thao tác trên tài nguyên được quản lý bởi ứng dụng

## Thiết lập môi trường của bạn

### Các điều kiện tiên quyết

Sau đây là các điều kiện tiên quyết cho việc thiết lập môi trường EKS của bạn, sau đó sẽ được quản lý bởi ArgoCD thông qua Amazon CLI.

- Một tài khoản AWS với quyền thích hợp

- AWS CLI phiên bản 2.13.0 hoặc mới hơn

- Node.js phiên bản 18.0.0 hoặc mới hơn

- npm phiên bản 9.0.0 hoặc mới hơn

- Amazon Q CLI phiên bản 1.0.0 hoặc mới hơn (`npm install -g @aws/amazon-q-cli`)

- Một cụm EKS (phiên bản 1.27 hoặc mới hơn) đã được cài ArgoCD phiên bản 2.8 hoặc mới hơn

### Kết nối với cụm EKS của bạn

1. Sử dụng AWS CLI để cập nhật kubeconfig của bạn

`aws eks update-kubeconfig --name <cluster_name> --region <region> --role-arn <iam_role_arn>`

2. Xác minh các pod ArgoCD đang chạy đúng cách trong namespace argocd

`kubectl get pods -n argocd`

3. Truy cập giao diện người dùng ArgoCD server trên máy cục bộ bằng lệnh chuyển tiếp port

`kubectl port-forward svc/blueprints-addon-argocd-server -n argocd 8080:443`

### Tạo ArgoCD API token

1. Truy cập vào **ArgoCD UI** tại [https://localhost:8080](https://localhost:8080)
2. Đăng nhập bằng thông tin đăng nhập của tài khoản admin
3. Điều hướng đến User **Settings > API Tokens**
4. Nhấp vào **“Generate New”** để tạo một token mới
5. Tạo một tệp cấu hình Amazon Q CLI MCP tại đường dẫn `.amazonq/mcp.json` và cập nhật các giá trị **ARGOCD_BASE_URL** và **ARGOCD_API_TOKEN** sao cho phù hợp với thiết lập môi trường của bạn.

### Tích hợp với Amazon Q CLI

```JSON
{
  "mcpServers": {
    "argocd-mcp-stdio": {
      "type": "stdio",
      "command": "npx",
      "args": [
         "argocd-mcp@latest",
         "stdio"
      ],
      "env": {
        "ARGOCD_BASE_URL": "<ARGOCD_BASE_URL>",
        "ARGOCD_API_TOKEN": "<ARGOCD_API_TOKEN>",
        "NODE_TLS_REJECT_UNAUTHORIZED": "0"
      }
    }
  }
}
```

Khi cấu hình xong, bạn có thể bắt đầu sử dụng câu lệnh bằng ngôn ngữ tự nhiên với Amazon Q CLI để tương tác với các ứng dụng ArgoCD của mình.

### Quản lý các ứng dụng bằng cách sử dụng ngôn ngữ tự nhiên

Dưới đây là một số câu lệnh (prompts) để tương tác với các ứng dụng ArgoCD trong cụm EKS của bạn

**Liệt kê ứng dụng Argo**

**Câu lệnh**: `List all ArgoCD applications in my cluster`

![example](../../../static/images/17632-image-8.png)

<p style="text-align: center;"><i>Amazon Q sẽ sử dụng máy chủ ArgoCD MCP để truy xuất và hiển thị tất cả các ứng dụng</i></p>

**Tạo ứng dụng ArgoCD mới**

**Câu lệnh**: `Create new argocd application using App name: game-2048   Repo: https://github.com/aws-ia/terraform-aws-eks-blueprints  Path: patterns/gitops/getting-started-argocd/k8s. Branch: main  Namespace: argocd`

![example](../../../images/Blog6/17632-image-2.png)

<p style="text-align: center;"><i>Amazon Q sẽ tạo một ứng dụng mới từ thông tin GitRepo cung cấp</i></p>

**Xem trạng thái triển khai**

**Câu lệnh**: `Show me the resource tree for team-carmen app`

![example](../../../static/images/17632-image-8.png)

<p style="text-align: center;"><i>Amazon Q sẽ hiển thị cấu trúc phân cấp của các tài nguyên Kubernetes do ứng dụng quản lý.</i></p>

**Đồng bộ hóa ứng dụng**

**Câu lệnh**: `Show me the applications that’s out of sync`

![example](../../../images/Blog6/17632-image-3.png)

<p style="text-align: center;"><i>Amazon Q sẽ hiển thị các ứng dụng không đồng bộ</i></p>

**Câu lệnh**: `Sync the application`

![example](../../../images/Blog6/17632-image-4.png)

<p style="text-align: center;"><i>Amazon Q đang đồng bộ hóa các ứng dụng</i></p>

Amazon Q sẽ:

- Khởi tạo hoạt động đồng bộ cho ứng dụng đã chỉ định
- Giám sát quá trình động bộ hóa
- Báo trạng thái cuối cùng của hoạt động đồng bộ hóa

**Kiểm tra sức khỏe và giám sát**

**Câu lệnh**: `Check the health of all resources in the team-geordie application`

![example](../../../images/Blog6/17632-image-6.png)

<p style="text-align: center;"><i>Amazon Q đang đưa ra trạng thái sức khỏe của tất cả tài nguyên trong một ứng dụng</i></p>

Amazon Q sẽ:

- Truy vấn trạng thái sức khỏe của tất cả tài nguyên
- Xác định bất cứ thành phần nào không lành mạnh
- Cung cấp các khuyến nghị để giải quyết các vấn đề

**Câu lệnh**: `Show me the logs for the failing pod in the team-platform application`

![example](../../../images/Blog6/17632-image-7.png)

<p style="text-align: center;"><i>Amazon Q đang hiển thị các log của pod gặp sự cố</i></p>

Amazon Q sẽ:

- Xác định các pod gặp sự cố
- Truy vấn và hiển thị các log có liên quan
- Làm nổi bật các thông báo lỗi tiềm ẩn

## Tổng kết

Việc tích hợp Amazon Q CLI với ArgoCD thông qua MCP server đánh dấu một bước tiến mang tính chuyển đổi trong việc quản lý Kubernets, kết hợp khả năng GitOps của ArgoCD với công nghệ xử lý ngôn ngữ tự nhiên của Amazon Q. Bằng cách chuyển đổi các thao tác phức tạp trên Kubernets thành các tương tác hội thoại đơn giản, giải pháp này cho phép các nhóm có thể tập trung vào điều thực sự quan trọng - tạo ra giá trị cho doanh nghiệp. Hơn là dành thời gian ghi nhớ các lệnh hoặc vật lộn với những kỹ thuật phức tạp, giờ đây các nhóm có thể quản lý hạ tầng cloud của họ thông qua đối thoại tự nhiên, giúp hành trình cloud-native trở nên dễ tiếp cận và hiệu quả hơn cho mọi người. Sẵn sàng cho việc chuyển đổi trải nghiệm EKS và ArgoCD của bạn chưa? Hãy thử tích hợp Amazon Q CLI với ArgoCD MCP ngay và khám phá lý do vì sao các đội DevOps đang đưa nó vào bộ công cụ thiết yếu của họ.
