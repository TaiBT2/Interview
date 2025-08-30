# DevOps Architect Interview Preparation Guide
## Phân tích câu hỏi phỏng vấn DevOps Architect tại Atlassian

### 📋 Tổng quan
Đây là bộ câu hỏi phỏng vấn cho vị trí **DevOps Architect** tại Atlassian, được chia thành 3 vòng với tổng thời gian 150 phút. Các câu hỏi tập trung vào kinh nghiệm thực tế, tư duy hệ thống và khả năng leadership.

---

## 🎯 Vòng 1: Infra, Kubernetes, và Cloud Patterns (45 phút)

### 1.1 Thiết kế cluster EKS multi-tenant

**Câu hỏi:** Làm sao để các team dev, QA, prod dùng chung một cluster mà không dính dáng gì tới nhau?

**Hướng tiếp cận:**
- **Namespace isolation** với ResourceQuotas và LimitRanges
- **Network policies** để kiểm soát traffic giữa các namespace
- **RBAC** với ServiceAccounts riêng biệt
- **Pod Security Standards** (PSS) để enforce security policies

**Gợi ý trả lời:**
```yaml
# Ví dụ ResourceQuota
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
```

**Ôn tập:**
- [Kubernetes Multi-tenancy](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- [RBAC Best Practices](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

**Câu trả lời hoàn thiện:**
"Để thiết kế cluster EKS multi-tenant cho các team dev, QA, prod, tôi sẽ áp dụng kiến trúc namespace-based isolation với các layer bảo mật:

**1. Namespace Isolation:**
- Tạo namespace riêng cho từng team: `dev-team`, `qa-team`, `prod-team`
- Sử dụng ResourceQuotas để giới hạn tài nguyên mỗi team có thể sử dụng
- Implement LimitRanges để enforce default resource limits

**2. Network Security:**
- Áp dụng Network Policies để kiểm soát traffic giữa các namespace
- Default deny-all policy, chỉ cho phép traffic cần thiết
- Sử dụng service mesh (Istio) để fine-grained traffic control

**3. RBAC & Access Control:**
- Tạo ServiceAccounts riêng cho từng team
- Implement least privilege principle với Role và RoleBinding
- Sử dụng OIDC integration với AWS IAM cho authentication

**4. Security Policies:**
- Enforce Pod Security Standards (PSS) với admission controllers
- Implement security scanning với tools như Falco
- Regular security audits và compliance checks

**5. Monitoring & Observability:**
- Separate logging và monitoring cho từng namespace
- Implement alerting khi có suspicious activities
- Regular backup và disaster recovery testing"

### 1.2 Quản lý Kustomize overlays

**Câu hỏi:** Khi có hơn 10 cái Kustomize overlays, làm thế nào để quản lý mà không bị drift hay duplicate?

**Hướng tiếp cận:**
- **Base configuration** chuẩn hóa
- **Common labels** và annotations
- **Kustomize transformers** để tự động hóa
- **GitOps workflow** với ArgoCD/Flux

**Gợi ý trả lời:**
```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- ../base

namespace: production

commonLabels:
  app: myapp
  environment: prod

patches:
- path: patches/production.yaml
  target:
    kind: Deployment
    name: myapp
```

**Ôn tập:**
- [Kustomize Best Practices](https://kustomize.io/)
- [GitOps Principles](https://www.gitops.tech/)

**Câu trả lời hoàn thiện:**
"Để quản lý hơn 10 Kustomize overlays mà không bị drift hay duplicate, tôi sẽ implement một strategy có cấu trúc:

**1. Base Configuration Strategy:**
- Tạo một base configuration chuẩn hóa với common resources
- Sử dụng common labels và annotations để tracking
- Implement consistent naming conventions

**2. Overlay Organization:**
```
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   ├── staging/
│   └── production/
```

**3. Drift Prevention:**
- Sử dụng GitOps workflow với ArgoCD/Flux
- Implement automated testing cho mỗi overlay
- Regular sync và validation checks

**4. Duplication Avoidance:**
- Sử dụng Kustomize transformers để tự động hóa
- Implement common patches và configurations
- Regular audit và cleanup processes

**5. Monitoring & Validation:**
- Automated testing cho configuration changes
- Regular drift detection và alerting
- Documentation và change tracking"

### 1.3 Bảo mật S3 replication cross-region

**Câu hỏi:** Giải thích cách bảo mật việc nhân bản dữ liệu S3 giữa các region khác nhau và làm sao để xác thực tính data integrity khi scale.

**Hướng tiếp cận:**
- **Server-side encryption** với KMS
- **Cross-region replication** với encryption
- **Checksum validation** và integrity checks
- **Access logging** và monitoring

**Gợi ý trả lời:**
```bash
# Kiểm tra replication status
aws s3api get-bucket-replication --bucket source-bucket

# Validate checksum
aws s3api head-object --bucket dest-bucket --key object-key --checksum-mode ENABLED
```

**Ôn tập:**
- [S3 Replication](https://docs.aws.amazon.com/AmazonS3/latest/userguide/replication.html)
- [S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)

**Câu trả lời hoàn thiện:**
"Để bảo mật S3 replication cross-region và đảm bảo data integrity khi scale, tôi sẽ implement một comprehensive security strategy:

**1. Encryption Strategy:**
- Server-side encryption với AWS KMS cho cả source và destination buckets
- Customer-managed keys (CMK) để full control over encryption
- Encrypt data in transit với TLS 1.2+

**2. Cross-Region Replication Setup:**
- Configure replication rules với encryption requirements
- Implement versioning cho both source và destination buckets
- Use bucket policies để enforce encryption

**3. Data Integrity Validation:**
- Implement checksum validation cho mỗi object
- Use S3 Inventory để track replication status
- Regular integrity checks với automated scripts

**4. Access Control & Monitoring:**
- Implement least privilege access với IAM policies
- Enable CloudTrail logging cho all S3 operations
- Set up CloudWatch alarms cho replication failures

**5. Disaster Recovery:**
- Regular backup testing và validation
- Implement cross-region failover procedures
- Document recovery time objectives (RTO) và recovery point objectives (RPO)"

### 1.4 Xử lý lỗi systemd trong container

**Câu hỏi:** Khi systemd trên một node container gặp lỗi, chuyện gì sẽ xảy ra? Bác sẽ tự động phục hồi nó như thế nào?

**Hướng tiếp cận:**
- **Container restart policies**
- **Health checks** và readiness probes
- **Liveness probes** để detect systemd failures
- **Node failure detection** và pod rescheduling

**Gợi ý trả lời:**
```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - name: app
        livenessProbe:
          exec:
            command:
            - systemctl
            - is-active
            - my-service
          initialDelaySeconds: 30
          periodSeconds: 10
      restartPolicy: Always
```

**Ôn tập:**
- [Container Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/)
- [Systemd in Containers](https://systemd.io/CONTAINER_INTERFACE/)

**Câu trả lời hoàn thiện:**
"Khi systemd trong container gặp lỗi, đây là comprehensive approach để handle và auto-recover:

**1. Root Cause Analysis:**
- Systemd failures thường xảy ra do resource exhaustion, configuration issues, hoặc kernel problems
- Container restart policies sẽ trigger automatic recovery
- Health checks sẽ detect service failures

**2. Automatic Recovery Strategy:**
- Implement liveness probes để detect systemd service failures
- Use readiness probes để ensure service is ready to serve traffic
- Configure appropriate restart policies (Always, OnFailure, Never)

**3. Monitoring & Alerting:**
- Monitor systemd service status với custom health checks
- Set up alerts cho service failures và restart events
- Track restart frequency để identify patterns

**4. Prevention Measures:**
- Proper resource limits để prevent OOM kills
- Regular health checks và monitoring
- Implement circuit breakers cho critical services

**5. Escalation Procedures:**
- Automated rollback nếu service fails repeatedly
- Manual intervention procedures cho critical services
- Documentation và runbooks cho common issues"

### 1.5 Phát hiện và ngăn chặn tấn công lateral movement

**Câu hỏi:** Chiến lược để detect và minimize việc một pod bị chiếm quyền và từ đó mitigate các pod khác trong cùng cluster.

**Hướng tiếp cận:**
- **Network policies** để limit pod-to-pod communication
- **Security scanning** với tools như Falco
- **Runtime security** monitoring
- **Pod security standards** enforcement

**Gợi ý trả lời:**
```yaml
# Network Policy example
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

**Ôn tập:**
- [Falco Security](https://falco.org/)
- [Kubernetes Security](https://kubernetes.io/docs/concepts/security/)

**Câu trả lời hoàn thiện:**
"Để detect và minimize lateral movement attacks trong Kubernetes cluster, tôi sẽ implement một defense-in-depth strategy:

**1. Network Segmentation:**
- Implement Network Policies để restrict pod-to-pod communication
- Use default deny-all policies, chỉ allow necessary traffic
- Segment workloads theo security zones và trust levels

**2. Runtime Security Monitoring:**
- Deploy Falco để detect suspicious activities
- Monitor for privilege escalation attempts
- Track unusual network connections và file access

**3. Access Control & RBAC:**
- Implement least privilege principle với fine-grained RBAC
- Use ServiceAccounts với minimal required permissions
- Regular access reviews và permission audits

**4. Security Scanning & Compliance:**
- Implement admission controllers để enforce security policies
- Use OPA (Open Policy Agent) để policy enforcement
- Regular vulnerability scanning với tools như Trivy

**5. Incident Response:**
- Automated alerting cho suspicious activities
- Incident response playbooks cho different attack scenarios
- Regular security drills và tabletop exercises

**6. Continuous Monitoring:**
- Real-time monitoring với Prometheus và Grafana
- Log aggregation và analysis với ELK stack
- Regular security assessments và penetration testing"

### 1.6 Upgrade zero-downtime cho stateful workload

**Câu hỏi:** Các bác sẽ dùng Helm 3 để nâng cấp một ứng dụng có trạng thái stateful mà không gây ra downtime cho service như thế nào?

**Hướng tiếp cận:**
- **Rolling updates** với StatefulSets
- **Helm hooks** để manage pre/post upgrade
- **Database migration** strategies
- **Backup và restore** procedures

**Gợi ý trả lời:**
```bash
# Helm upgrade với atomic rollback
helm upgrade myapp ./chart --atomic --timeout 10m --wait

# StatefulSet rolling update
kubectl patch statefulset myapp -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'
```

**Ôn tập:**
- [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)

**Câu trả lời hoàn thiện:**
"Để upgrade stateful workload với zero-downtime sử dụng Helm 3, tôi sẽ implement một comprehensive strategy:

**1. Pre-Upgrade Preparation:**
- Backup critical data và state
- Test upgrade process trong staging environment
- Review và validate Helm chart changes
- Ensure rollback procedures are ready

**2. Helm Upgrade Strategy:**
- Use `--atomic` flag để automatic rollback nếu upgrade fails
- Set appropriate `--timeout` và `--wait` parameters
- Implement `--dry-run` để validate changes before actual upgrade
- Use `--reuse-values` để preserve existing configurations

**3. StatefulSet Considerations:**
- Configure `updateStrategy.type: RollingUpdate` cho StatefulSets
- Set appropriate `partition` để control update order
- Ensure proper volume claim templates và persistent storage
- Monitor pod readiness và health during upgrade

**4. Database Migration (if applicable):**
- Implement database migration scripts
- Use database versioning và migration tools
- Test data integrity after migration
- Have rollback procedures cho database changes

**5. Monitoring & Validation:**
- Monitor application health và performance metrics
- Validate data consistency và integrity
- Check service endpoints và connectivity
- Monitor resource usage và performance

**6. Rollback Procedures:**
- Document rollback steps và procedures
- Test rollback process trong staging
- Have backup và restore procedures ready
- Monitor post-rollback system health"

### 1.7 Kiến trúc hybrid cloud routing

**Câu hỏi:** Mô tả một kiến trúc routing giữa GCP và AWS. Bác sẽ đặt các enforce boundaries ở đâu để kiểm soát traffic và security?

**Hướng tiếp cận:**
- **VPN/Direct Connect** connections
- **Load balancers** với health checks
- **Security groups** và firewall rules
- **Traffic routing** policies

**Gợi ý trả lời:**
```
AWS VPC <-> VPN Gateway <-> Cloud Router <-> GCP VPC
     |                           |
Load Balancer              Load Balancer
     |                           |
Security Groups           Firewall Rules
```

**Ôn tập:**
- [AWS VPN](https://docs.aws.amazon.com/vpn/)
- [GCP Cloud Router](https://cloud.google.com/network-connectivity/docs/router)

**Câu trả lời hoàn thiện:**
"Để thiết kế kiến trúc routing giữa GCP và AWS với proper security boundaries, tôi sẽ implement một hybrid cloud architecture:

**1. Network Connectivity:**
- AWS VPC <-> VPN Gateway <-> Cloud Router <-> GCP VPC
- Use AWS Direct Connect hoặc VPN connections cho high bandwidth
- Implement redundant connections để ensure availability
- Configure appropriate routing tables và route propagation

**2. Load Balancing Strategy:**
- Deploy Application Load Balancers (ALB) trên AWS
- Use Google Cloud Load Balancer trên GCP
- Implement health checks và failover mechanisms
- Configure cross-region load balancing nếu needed

**3. Security Boundaries:**
- Implement Security Groups trên AWS để control traffic
- Use Firewall Rules trên GCP để restrict access
- Deploy Network Security Groups cho additional protection
- Implement WAF (Web Application Firewall) cho both clouds

**4. Traffic Management:**
- Use Route 53 cho DNS management và health checks
- Implement traffic routing policies với weighted routing
- Configure failover routing cho disaster recovery
- Monitor traffic patterns và performance metrics

**5. Monitoring & Observability:**
- Centralized logging với CloudWatch và Stackdriver
- Implement distributed tracing với Jaeger hoặc Zipkin
- Monitor network latency và bandwidth utilization
- Set up alerts cho connectivity issues

**6. Compliance & Governance:**
- Implement consistent security policies across clouds
- Regular security audits và compliance checks
- Document network architecture và security controls
- Implement change management procedures"

### 1.8 Khôi phục Terraform state

**Câu hỏi:** Chẳng may file state của Terraform bị corrupt trong lúc đang migration backend. Chiến lược rebuild của bác là gì?

**Hướng tiếp cận:**
- **State backup** strategies
- **Remote state** storage (S3, GCS)
- **State import** procedures
- **Workspace management**

**Gợi ý trả lời:**
```bash
# Import existing resources
terraform import aws_instance.web i-12345678

# Recreate state from existing infrastructure
terraform plan -refresh-only
terraform apply -auto-approve
```

**Ôn tập:**
- [Terraform State Management](https://www.terraform.io/docs/language/state/)
- [Terraform Import](https://www.terraform.io/docs/cli/import/)

**Câu trả lời hoàn thiện:**
"Khi Terraform state bị corrupt trong quá trình migration backend, đây là comprehensive strategy để rebuild:

**1. Immediate Assessment:**
- Identify the extent of corruption và affected resources
- Check if backup state files are available
- Assess impact on running infrastructure
- Determine if partial recovery is possible

**2. Backup & Recovery Strategy:**
- Use remote state storage (S3, GCS) với versioning enabled
- Implement automated state backups với retention policies
- Use Terraform workspaces để isolate environments
- Regular state validation và integrity checks

**3. State Rebuild Process:**
- Use `terraform import` để import existing resources
- Implement `terraform plan -refresh-only` để sync state
- Use `terraform state list` để inventory existing resources
- Document all imported resources và their configurations

**4. Validation & Testing:**
- Run `terraform plan` để verify state consistency
- Test resource operations trong non-production environment
- Validate resource configurations và dependencies
- Ensure no resource conflicts hoặc duplicates

**5. Prevention Measures:**
- Implement state locking với DynamoDB (AWS) hoặc Cloud Storage (GCP)
- Use Terraform Cloud hoặc Enterprise cho enhanced state management
- Regular state backups và disaster recovery testing
- Implement change management procedures

**6. Documentation & Runbooks:**
- Document recovery procedures và lessons learned
- Create runbooks cho common state issues
- Train team members on state management best practices
- Regular review và update of procedures"

### 1.9 Bash One-liner

**Câu hỏi:** Viết một dòng lệnh Bash để tìm tất cả các container đang chạy và sử dụng memory RSS lớn hơn 500MB trên một node.

**Gợi ý trả lời:**
```bash
docker stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}" | awk 'NR>1 && $2+0>500'
```

**Ôn tập:**
- [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/)
- [Bash Scripting](https://www.gnu.org/software/bash/manual/)

**Câu trả lời hoàn thiện:**
"Để tìm tất cả container đang chạy và sử dụng memory RSS > 500MB, tôi sẽ sử dụng một combination của Docker commands và text processing:

**1. Docker Stats Command:**
```bash
docker stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}\t{{.MemPerc}}"
```

**2. Advanced One-liner với awk:**
```bash
docker stats --no-stream --format "{{.Container}}\t{{.MemUsage}}" | awk 'NR>1 {gsub(/[A-Za-z]/,"",$2); if($2+0>500) print $1 " uses " $2 "MB"}'
```

**3. Alternative với jq (nếu available):**
```bash
docker stats --no-stream --format json | jq -r '.[] | select(.MemUsage | tonumber > 500) | "\(.Container) uses \(.MemUsage)"'
```

**4. Comprehensive Script:**
```bash
#!/bin/bash
echo "Containers using more than 500MB RSS:"
docker stats --no-stream --format "table {{.Container}}\t{{.MemUsage}}\t{{.MemPerc}}\t{{.CPUPerc}}" | \
awk 'NR>1 {
    gsub(/[A-Za-z]/,"",$2)
    if($2+0>500) {
        printf "%-30s %-15s %-10s %-10s\n", $1, $2"MB", $3, $4
    }
}'
```

**5. Monitoring Script với Alerting:**
```bash
#!/bin/bash
THRESHOLD=500
ALERT_EMAIL="admin@company.com"

HIGH_MEMORY_CONTAINERS=$(docker stats --no-stream --format "{{.Container}}\t{{.MemUsage}}" | \
awk -v threshold=$THRESHOLD 'NR>1 {
    gsub(/[A-Za-z]/,"",$2)
    if($2+0>threshold) print $1 ":" $2 "MB"
}')

if [ ! -z "$HIGH_MEMORY_CONTAINERS" ]; then
    echo "High memory usage detected:" | mail -s "Container Memory Alert" $ALERT_EMAIL
    echo "$HIGH_MEMORY_CONTAINERS" | mail -s "Container Memory Alert" $ALERT_EMAIL
fi
```

**Key Points:**
- Use `--no-stream` để get single snapshot
- Parse memory values với text processing
- Handle different memory formats (MB, GB, etc.)
- Consider performance impact trên large deployments
- Implement proper error handling và logging"

---

## 🔥 Vòng 2: Troubleshooting & RCA (75 phút)

### 2.1 Lỗi TLS handshake trên ALB

**Câu hỏi:** Một config mới trên AWS ALB gây ra lỗi TLS handshake lúc được lúc không. Trình bày toàn bộ quá trình RCA.

**Quy trình RCA:**
1. **Collect logs** từ ALB access logs
2. **Check SSL/TLS configuration** trên ALB
3. **Verify certificate** validity và chain
4. **Test connectivity** từ different clients
5. **Check security groups** và network ACLs

**Gợi ý trả lời:**
```bash
# Check ALB logs
aws logs filter-log-events --log-group-name /aws/applicationloadbalancer/alb-name

# Test SSL connection
openssl s_client -connect alb-endpoint:443 -servername domain.com

# Check certificate
aws acm describe-certificate --certificate-arn arn:aws:acm:region:account:certificate/cert-id
```

**Ôn tập:**
- [ALB Troubleshooting](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-troubleshooting.html)
- [SSL/TLS Debugging](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/create-https-listener.html)

**Câu trả lời hoàn thiện:**
"Để troubleshoot lỗi TLS handshake trên ALB, tôi sẽ follow một systematic RCA process:

**1. Initial Assessment:**
- Check ALB access logs để identify patterns
- Verify if issue affects all clients hoặc specific ones
- Determine if problem is intermittent hoặc consistent
- Check recent configuration changes

**2. SSL/TLS Configuration Analysis:**
- Verify SSL certificate validity và expiration
- Check certificate chain completeness
- Validate SSL policy configuration
- Review security group rules cho port 443

**3. Client-Side Testing:**
- Test SSL connection với different clients
- Use openssl để verify certificate chain
- Check browser compatibility issues
- Test với different TLS versions

**4. Network & Infrastructure Check:**
- Verify ALB health status
- Check target group health
- Review VPC configuration và routing
- Validate security groups và NACLs

**5. Log Analysis:**
- Analyze CloudWatch logs cho ALB
- Check application logs cho backend issues
- Review CloudTrail logs cho configuration changes
- Monitor metrics cho performance degradation

**6. Resolution Steps:**
- Update SSL certificate nếu expired
- Adjust SSL policy nếu compatibility issues
- Modify security group rules nếu needed
- Implement proper monitoring và alerting

**7. Prevention Measures:**
- Set up certificate expiration alerts
- Implement automated certificate renewal
- Regular SSL configuration audits
- Document troubleshooting procedures"

### 2.2 kubectl logs không ra gì

**Câu hỏi:** Các node k8s vẫn báo "healthy", nhưng lệnh kubectl logs vào các pod quan trọng thì lại trống trơn.

**Quy trình debug:**
1. **Check pod status** và container state
2. **Verify logging configuration** (stdout/stderr)
3. **Check kubelet logs** trên node
4. **Verify container runtime** logs
5. **Check storage** và disk space

**Gợi ý trả lời:**
```bash
# Check pod details
kubectl describe pod pod-name -n namespace

# Check kubelet logs
journalctl -u kubelet -f

# Check container logs directly
docker logs container-id
```

**Ôn tập:**
- [Kubernetes Logging](https://kubernetes.io/docs/concepts/cluster-administration/logging/)
- [Troubleshooting Pods](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)

**Câu trả lời hoàn thiện:**
"Khi kubectl logs không ra gì dù node healthy, đây là systematic approach để debug:

**1. Pod & Container State Analysis:**
- Check pod status với `kubectl describe pod`
- Verify container state và restart count
- Check if containers are actually running
- Review pod events và error messages

**2. Logging Configuration Verification:**
- Verify application is writing to stdout/stderr
- Check if logging driver is properly configured
- Review container runtime logging settings
- Validate log rotation policies

**3. Node-Level Investigation:**
- Check kubelet logs với `journalctl -u kubelet`
- Verify container runtime (Docker/containerd) status
- Check node disk space và inode usage
- Review node resource constraints

**4. Container Runtime Debugging:**
- Access container directly với `kubectl exec`
- Check container logs với `docker logs` hoặc `crictl logs`
- Verify container filesystem và permissions
- Check if application is actually running

**5. Storage & Filesystem Check:**
- Verify log storage location và permissions
- Check for disk space issues
- Review log rotation và retention policies
- Validate filesystem integrity

**6. Network & Connectivity:**
- Check network connectivity từ node to logging infrastructure
- Verify logging service endpoints
- Review firewall rules và security policies
- Test logging pipeline connectivity

**7. Resolution Steps:**
- Restart problematic containers nếu needed
- Adjust resource limits nếu constrained
- Fix logging configuration issues
- Implement proper monitoring và alerting

**8. Prevention Measures:**
- Implement comprehensive logging strategy
- Set up log monitoring và alerting
- Regular log storage maintenance
- Document troubleshooting procedures"

### 2.3 Sidecar agent gây CPU throttling

**Câu hỏi:** Bác vừa deploy một con sidecar để ghi log, tự nhiên CPU throttling tăng đột biến.

**Quy trình debug:**
1. **Check resource limits** của sidecar
2. **Monitor CPU usage** với metrics
3. **Analyze sidecar logs** và configuration
4. **Check for infinite loops** hoặc inefficient processing
5. **Consider resource requests** adjustment

**Gợi ý trả lời:**
```bash
# Check resource usage
kubectl top pod pod-name -n namespace

# Check sidecar logs
kubectl logs pod-name -c sidecar-container -n namespace

# Monitor CPU throttling
kubectl exec pod-name -c sidecar-container -- cat /sys/fs/cgroup/cpu/cpu.stat
```

**Ôn tập:**
- [Kubernetes Resource Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [Sidecar Pattern](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns/)

### 2.4 Autoscaling không hoạt động

**Câu hỏi:** CPU đã vượt ngưỡng từ lâu mà autoscaling không thèm "nhảy".

**Quy trình debug:**
1. **Check HPA configuration** và metrics
2. **Verify metrics server** health
3. **Check resource requests** và limits
4. **Monitor scaling events** và cooldown periods
5. **Verify node resources** availability

**Gợi ý trả lời:**
```bash
# Check HPA status
kubectl describe hpa hpa-name -n namespace

# Check metrics server
kubectl get --raw /apis/metrics.k8s.io/v1beta1/pods

# Check scaling events
kubectl get events --sort-by='.lastTimestamp' -n namespace
```

**Ôn tập:**
- [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
- [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)

### 2.5 Lỗi 504 Gateway Timeout

**Câu hỏi:** User cuối báo lỗi 504 liên tục, nhưng health check của ELB vẫn báo "xanh lè".

**Quy trình debug:**
1. **Check application logs** và response times
2. **Verify health check configuration** (path, timeout, interval)
3. **Monitor backend instances** health
4. **Check network connectivity** và latency
5. **Analyze application performance** bottlenecks

**Gợi ý trả lời:**
```bash
# Check ELB health
aws elbv2 describe-target-health --target-group-arn tg-arn

# Check application logs
kubectl logs deployment/app -n namespace --tail=100

# Test endpoint directly
curl -v --connect-timeout 10 --max-time 30 http://app-endpoint/health
```

**Ôn tập:**
- [ELB Troubleshooting](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-troubleshooting.html)
- [Application Performance Monitoring](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-usage-monitoring/)

### 2.6 Mất log của systemd journal

**Câu hỏi:** Log của systemd journal trên một vài AMI biến mất sạch sau khi reboot.

**Quy trình debug:**
1. **Check journald configuration** (Storage, MaxRetentionSec)
2. **Verify disk space** và inode usage
3. **Check journal corruption** và recovery
4. **Review AMI build process** và configuration
5. **Implement persistent logging** strategies

**Gợi ý trả lời:**
```bash
# Check journald config
cat /etc/systemd/journald.conf

# Check disk space
df -h /var/log/journal

# Recover corrupted journal
journalctl --verify
journalctl --sync
```

**Ôn tập:**
- [Systemd Journal](https://www.freedesktop.org/software/systemd/man/journald.conf.html)
- [AMI Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-best-practices.html)

### 2.7 Pod bị OOMKilled nhưng không có log

**Câu hỏi:** Một pod quan trọng ở môi trường production bị "OOMKilled" nhưng không tìm thấy một dòng log nào.

**Quy trình forensic:**
1. **Check node events** và system logs
2. **Analyze memory usage** patterns
3. **Review resource limits** và requests
4. **Check for memory leaks** trong application
5. **Implement memory monitoring** và alerting

**Gợi ý trả lời:**
```bash
# Check node events
kubectl get events --field-selector involvedObject.name=pod-name -n namespace

# Check system logs
journalctl -u kubelet | grep -i oom

# Analyze memory usage
kubectl top pod pod-name -n namespace --containers
```

**Ôn tập:**
- [Memory Management](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)
- [OOM Debugging](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)

### 2.8 Kernel panic trên node GKE

**Câu hỏi:** Một node trên GKE bị "kernel panic" ngay giữa lúc đang deploy.

**Quy trình debug:**
1. **Check node status** và conditions
2. **Review kernel logs** và crash dumps
3. **Analyze resource usage** và hardware health
4. **Check for known issues** với GKE version
5. **Implement node monitoring** và alerting

**Gợi ý trả lời:**
```bash
# Check node status
kubectl describe node node-name

# Check kernel logs
gcloud compute ssh node-name --command="dmesg | tail -50"

# Check GKE version
kubectl get nodes -o wide
```

**Ôn tập:**
- [GKE Troubleshooting](https://cloud.google.com/kubernetes-engine/docs/troubleshooting)
- [Kernel Debugging](https://www.kernel.org/doc/html/latest/admin-guide/bug-hunting.html)

---

## 🎯 Vòng 3: Leadership & Strategy (30 phút)

### 3.1 Cấp quyền cho Developer

**Câu hỏi:** Làm thế nào để thiết kế một hạ tầng vừa cấp quyền cho dev tự do sáng tạo, vừa không cung cấp cho họ quyền kiểm soát?

**Hướng tiếp cận:**
- **Self-service platforms** (Backstage, Port)
- **RBAC với least privilege principle**
- **Infrastructure as Code** với approval workflows
- **Monitoring và alerting** cho compliance

**Gợi ý trả lời:**
```yaml
# RBAC example
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: developer-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "create", "update"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
```

**Ôn tập:**
- [Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- [DevOps Culture](https://www.atlassian.com/devops)

**Câu trả lời hoàn thiện:**
"Để thiết kế hạ tầng vừa cấp quyền cho dev tự do sáng tạo vừa không cung cấp quyền kiểm soát, tôi sẽ implement một balanced approach:

**1. Self-Service Platform Strategy:**
- Deploy Backstage hoặc Port để provide developer portal
- Implement Infrastructure as Code templates với approval workflows
- Create standardized service catalogs cho common resources
- Provide automated provisioning với guardrails

**2. RBAC với Least Privilege Principle:**
- Design granular roles cho different developer levels
- Implement namespace-based access control
- Use ServiceAccounts với minimal required permissions
- Regular access reviews và permission audits

**3. Infrastructure as Code với Governance:**
- Implement GitOps workflow với ArgoCD/Flux
- Require pull request reviews cho infrastructure changes
- Use policy-as-code với OPA để enforce standards
- Automated testing và validation cho all changes

**4. Monitoring & Compliance:**
- Implement comprehensive logging và monitoring
- Set up alerts cho unauthorized access attempts
- Regular security audits và compliance checks
- Automated policy enforcement và violation detection

**5. Developer Experience Optimization:**
- Provide clear documentation và runbooks
- Implement automated testing và validation
- Create self-service troubleshooting tools
- Regular training và knowledge sharing sessions

**6. Security & Risk Management:**
- Implement network segmentation và isolation
- Use secrets management với Vault hoặc AWS Secrets Manager
- Regular vulnerability scanning và patching
- Incident response procedures và escalation paths

**7. Continuous Improvement:**
- Regular feedback collection từ development teams
- Performance metrics và optimization
- Technology evaluation và adoption
- Process refinement và automation

**Key Success Factors:**
- Balance security với developer productivity
- Clear communication và documentation
- Automated processes để reduce friction
- Regular review và improvement cycles"

### 3.2 Checklist cho custom AMI

**Câu hỏi:** Trước khi duyệt một custom AMI lên production, checklist ở tầng Linux của bác gồm những gì?

**Checklist:**
- **Security patches** và updates
- **Hardened configuration** (SSH, firewall)
- **Monitoring agents** installation
- **Logging configuration** setup
- **Performance tuning** và optimization

**Gợi ý trả lời:**
```bash
# Security checklist
sudo yum update -y
sudo systemctl enable auditd
sudo systemctl enable fail2ban

# Monitoring setup
sudo yum install -y cloudwatch-agent
sudo systemctl enable cloudwatch-agent

# Logging configuration
sudo mkdir -p /var/log/application
sudo chown app:app /var/log/application
```

**Ôn tập:**
- [AMI Security Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-best-practices.html)
- [Linux Hardening](https://www.cisecurity.org/benchmark/amazon_linux/)

### 3.3 Chuyển đổi sang Service-Mesh

**Câu hỏi:** Bác được yêu cầu chuyển từ mô hình logging tập trung sang mô hình observability dựa trên service-mesh.

**Phân tích:**
- **Pros:** Distributed tracing, automatic service discovery, traffic management
- **Cons:** Complexity, performance overhead, learning curve
- **Migration strategy:** Gradual rollout, feature flags, rollback plan

**Gợi ý trả lời:**
```yaml
# Istio example
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.example.com
  http:
  - route:
    - destination:
        host: myapp-service
        port:
          number: 8080
```

**Ôn tập:**
- [Istio Documentation](https://istio.io/docs/)
- [Service Mesh Architecture](https://www.nginx.com/blog/service-mesh-architecture/)

### 3.4 Chaos Engineering

**Câu hỏi:** Bác mô phỏng chaos ở production trong môi trường staging cho kubernetes như thế nào?

**Hướng tiếp cận:**
- **Chaos Monkey** cho pod termination
- **Network latency** và packet loss simulation
- **Resource exhaustion** testing
- **Gradual rollout** với monitoring

**Gợi ý trả lời:**
```yaml
# Chaos Mesh example
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure
spec:
  action: pod-failure
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      app: myapp
  duration: "30s"
```

**Ôn tập:**
- [Chaos Engineering](https://principlesofchaos.org/)
- [Chaos Mesh](https://chaos-mesh.org/)

### 3.5 Bảo vệ SLOs

**Câu hỏi:** Khi các chỉ số SLOs mà bác đặt ra có nguy cơ làm chậm tiến độ của dự án và bị leader phản đối, bác sẽ xử lý ra sao?

**Hướng tiếp cận:**
- **Data-driven approach** với metrics và evidence
- **Risk assessment** và impact analysis
- **Stakeholder communication** và education
- **Gradual implementation** với measurable improvements

**Gợi ý trả lời:**
```
1. Present current performance metrics
2. Show correlation between SLOs and business outcomes
3. Propose phased implementation approach
4. Demonstrate ROI and risk mitigation
5. Establish monitoring and feedback loops
```

**Ôn tập:**
- [SLO Best Practices](https://sre.google/sre-book/service-level-objectives/)
- [Stakeholder Management](https://www.pmi.org/learning/library/stakeholder-management-project-success-11775)

---

## 📚 Hướng dẫn ôn tập

### 🎯 Kiến thức cốt lõi cần nắm vững:

#### **Kubernetes Deep Dive:**
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Kubernetes in Action](https://www.manning.com/books/kubernetes-in-action)
- [CKA/CKAD Certification](https://www.cncf.io/certification/cka/)

#### **Cloud Platforms:**
- [AWS Solutions Architect](https://aws.amazon.com/certification/certified-solutions-architect-associate/)
- [GCP Professional Cloud Architect](https://cloud.google.com/certification/cloud-architect)
- [Azure Solutions Architect](https://docs.microsoft.com/en-us/certifications/azure-solutions-architect/)

#### **Infrastructure as Code:**
- [Terraform Documentation](https://www.terraform.io/docs/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Helm Charts](https://helm.sh/docs/chart_template_guide/)

#### **Security & Compliance:**
- [CIS Benchmarks](https://www.cisecurity.org/benchmark/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Kubernetes Security](https://kubernetes.io/docs/concepts/security/)

#### **Monitoring & Observability:**
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Best Practices](https://grafana.com/docs/grafana/latest/best-practices/)
- [Jaeger Tracing](https://www.jaegertracing.io/docs/)

### 🛠️ Tools cần thành thạo:

#### **Container & Orchestration:**
- Docker, containerd, CRI-O
- Kubernetes (kubectl, kustomize, helm)
- OpenShift, Rancher

#### **Cloud & Infrastructure:**
- AWS CLI, AWS CDK
- GCP CLI, Terraform
- Azure CLI, ARM Templates

#### **Monitoring & Logging:**
- Prometheus, Grafana
- ELK Stack, Fluentd
- Jaeger, Zipkin

#### **Security & Compliance:**
- Falco, OPA
- Vault, AWS Secrets Manager
- Trivy, Snyk

### 📖 Sách tham khảo:

1. **Site Reliability Engineering** - Google
2. **The Phoenix Project** - Gene Kim
3. **Kubernetes in Action** - Marko Lukša
4. **Terraform: Up & Running** - Yevgeniy Brikman
5. **Building Microservices** - Sam Newman

### 🎓 Khóa học online:

1. **Linux Academy** - Kubernetes Deep Dive
2. **A Cloud Guru** - AWS Solutions Architect
3. **Coursera** - Cloud Architecture
4. **edX** - DevOps Engineering
5. **Pluralsight** - Infrastructure as Code

### 🔧 Labs thực hành:

1. **Kubernetes the Hard Way** - Kelsey Hightower
2. **AWS Well-Architected Labs**
3. **GCP Architecture Framework**
4. **Azure Architecture Center**

---

## 🎯 Chiến lược ôn tập

### **Giai đoạn 1: Nền tảng (4-6 tuần)**
- Nắm vững Kubernetes concepts và operations
- Thành thạo ít nhất 1 cloud platform (AWS/GCP/Azure)
- Học Infrastructure as Code với Terraform

### **Giai đoạn 2: Nâng cao (4-6 tuần)**
- Deep dive vào security và compliance
- Học monitoring và observability
- Thực hành troubleshooting scenarios

### **Giai đoạn 3: Leadership (2-3 tuần)**
- Học về team management và communication
- Understanding business impact và ROI
- Practice presentation và negotiation skills

### **Giai đoạn 4: Mock Interview (1-2 tuần)**
- Practice với real scenarios
- Time management cho từng vòng
- Feedback và improvement

---

## 📝 Tips cho buổi phỏng vấn

### **Trước phỏng vấn:**
- Review lại các projects đã làm
- Prepare specific examples cho mỗi scenario
- Research về Atlassian culture và values

### **Trong phỏng vấn:**
- **STAR method** cho behavioral questions
- **Show your thinking process** cho technical questions
- **Ask clarifying questions** khi cần thiết
- **Be honest** về limitations và learning areas

### **Sau phỏng vấn:**
- Send thank you email
- Reflect on performance
- Identify areas for improvement

---

## 🎯 Kết luận

Đây là một bộ câu hỏi cực kỳ chất lượng, đòi hỏi:
- **Deep technical knowledge** về cloud và Kubernetes
- **Real-world experience** với complex systems
- **Leadership skills** và strategic thinking
- **Continuous learning** mindset

**Thời gian chuẩn bị:** 3-6 tháng tùy theo background hiện tại

**Success rate:** Thường chỉ 5-10% candidates pass được tất cả vòng

**Key success factors:**
- Hands-on experience với production systems
- Strong problem-solving skills
- Excellent communication abilities
- Business acumen và strategic thinking

---

*📚 Tài liệu này được tạo dựa trên bài viết từ [DevOps VietNam](https://devops.vn/posts/phong-van-devops-architect-tai-atlassian-khoai-den-muc-nao/) và kinh nghiệm thực tế trong lĩnh vực DevOps/Cloud Architecture.*
