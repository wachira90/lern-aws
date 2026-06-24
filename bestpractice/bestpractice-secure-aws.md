## แนวทางการออกแบบ AWS Infrastructure ให้ปลอดภัย

### 1. 🏗️ Network Architecture (Zero Trust)

**VPC Design**
- แบ่ง Subnet เป็น Public / Private / Database tier
- ใช้ **Private Subnet** สำหรับ workload หลักทั้งหมด
- วาง EC2, RDS, Lambda ใน Private subnet เสมอ
- ใช้ **NAT Gateway** สำหรับ outbound traffic เท่านั้น

**Security Groups & NACLs**
- ใช้หลัก **Least Privilege** — เปิดเฉพาะ port ที่จำเป็น
- ห้ามใช้ `0.0.0.0/0` บน inbound ยกเว้น ALB/CloudFront
- ใช้ **NACLs** เป็น stateless layer เพิ่มเติม

---

### 2. 🔑 Identity & Access Management (IAM)

- ใช้ **IAM Roles** แทน Access Keys เสมอ
- เปิด **MFA** ทุก account โดยเฉพาะ root
- ใช้ **Permission Boundary** จำกัดสิทธิ์สูงสุด
- ตรวจสอบ IAM Access Analyzer สม่ำเสมอ
- หลีกเลี่ยง inline policy — ใช้ managed policy แทน

---

### 3. 🔐 Data Protection

| Layer | วิธีการ |
|-------|--------|
| **At Rest** | KMS encryption สำหรับ S3, EBS, RDS, DynamoDB |
| **In Transit** | TLS 1.2+ บังคับทุก endpoint |
| **Secrets** | ใช้ **AWS Secrets Manager** / Parameter Store |
| **Backup** | เปิด versioning + MFA Delete บน S3 |

---

### 4. 🛡️ Threat Detection & Monitoring

- **AWS GuardDuty** — ตรวจจับ anomaly และ threat อัตโนมัติ
- **AWS Security Hub** — รวม findings จากทุก service
- **CloudTrail** — log ทุก API call (เปิดทุก region)
- **AWS Config** — ตรวจสอบ compliance อัตโนมัติ
- **VPC Flow Logs** — วิเคราะห์ traffic pattern

---

### 5. 🌐 Edge & Application Security

- ใช้ **CloudFront** + **AWS WAF** หน้า ALB ทุกครั้ง
- เปิด **AWS Shield Standard** (ฟรี) / Advanced สำหรับ DDoS
- ใช้ **ACM** สำหรับ SSL/TLS certificate (ต่ออายุอัตโนมัติ)
- เปิด **AWS Shield Advanced** ถ้ามี critical workload

---

### 6. 🏛️ Multi-Account Strategy

```
Root Account (ห้ามใช้งาน)
├── Security Account     → GuardDuty Master, Security Hub
├── Log Archive Account  → CloudTrail logs ทั้งหมด
├── Production Account   → Workload จริง
├── Staging Account      → Pre-production
└── Dev Account          → Development
```

ใช้ **AWS Organizations** + **Service Control Policies (SCP)** ควบคุมทุก account

---

### 7. ♻️ Incident Response & Recovery

- กำหนด **RTO / RPO** ให้ชัดเจน
- เปิด **AWS Backup** cross-region
- ทำ **Disaster Recovery drill** อย่างน้อยปีละครั้ง
- ใช้ **EventBridge + Lambda** สำหรับ automated remediation

---

### 8. ✅ Compliance & Best Practices

- รัน **AWS Trusted Advisor** ตรวจสุขภาพสม่ำเสมอ
- ใช้ **AWS Well-Architected Tool** ประเมิน Security Pillar
- ทำ **Penetration Testing** ตาม AWS policy
- ใช้ **Infrastructure as Code** (Terraform/CDK) เพื่อ consistency

---

### สรุป Priority การเริ่มต้น

1. **ทันที** → MFA, ปิด root access, เปิด CloudTrail
2. **สัปดาห์แรก** → VPC design, Security Groups, GuardDuty
3. **เดือนแรก** → Secrets Manager, WAF, Security Hub
4. **ระยะยาว** → Multi-account, Automation, DR drill
