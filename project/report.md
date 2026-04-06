# Empirical Analysis of SSH Security: Brute-Force Attack Resistance Under Varying Configurations

---

## Abstract

This report presents an empirical study on the security of SSH (Secure Shell) authentication under different password strength configurations and defensive mechanisms. The experiment was conducted in a controlled laboratory environment using Docker containers and a Rocky Linux virtual machine. Three configurations were tested: a weak password with no protection, a strong password with no protection, and a strong password combined with fail2ban intrusion prevention. Brute-force attacks were executed using Hydra, a well-known password-cracking tool, while Nmap was used for initial reconnaissance. Results demonstrated that weak passwords are trivially compromised within seconds, while strong passwords of sufficient length and complexity render brute-force attacks computationally infeasible. The introduction of fail2ban provided an additional layer of defense by actively blocking repeated authentication attempts, effectively halting automated attacks before completion. A mathematical analysis of the keyspace for a 16-character password using the full printable ASCII set confirmed that exhaustive brute-force would require approximately 1.39 × 10^18 years even at one million attempts per second. The findings reinforce established cybersecurity best practices regarding password policy, defense-in-depth strategies, and the critical importance of rate-limiting mechanisms in protecting network services.

---

## 1. Introduction

Secure Shell (SSH) is a widely used cryptographic protocol for secure remote administration of systems and services. It is a cornerstone of modern IT infrastructure, enabling encrypted communication between clients and servers. However, SSH services that rely on password-based authentication remain susceptible to brute-force attacks, in which an adversary systematically attempts large numbers of credential combinations to gain unauthorized access (Ylonen & Lonvick, 2006).

Brute-force attacks against SSH are among the most common threats observed in internet-facing systems. Automated tools such as Hydra and Medusa enable attackers to test thousands of password combinations per minute with minimal effort. The success of such attacks is primarily determined by two factors: the strength of the target password and the presence of defensive countermeasures such as account lockout policies or intrusion prevention systems.

The objective of this experiment was to empirically evaluate the resilience of an SSH server under three distinct configurations:

1. A weak password with no defensive protections.
2. A strong password with no defensive protections.
3. A strong password combined with fail2ban rate limiting.

By collecting data on attack success, duration, and number of attempts, this study aimed to provide evidence-based recommendations for securing SSH services in organizational environments.

---

## 2. Methodology

### 2.1 Environment Setup

The initial experimental environment consisted of a preconfigured SSH server running inside a Docker container, launched using Docker Compose. The container exposed SSH on port 2222 of the host machine. The default credentials for the base image were username `student` with password `password123`.

Three additional user accounts were created within the container using a provisioning script (`create_users.sh`), each representing a different password strength tier:

| User         | Password           | Classification |
|--------------|--------------------|----------------|
| weakuser     | 123456             | Weak           |
| mediumuser   | Password123        | Moderate       |
| stronguser   | Str0ng!Pass#2026   | Strong         |

It was noted during setup that the Docker image presented issues with service management, which prevented the installation of fail2ban within the container. To address this limitation, a separate Rocky Linux virtual machine was provisioned to host an SSH daemon alongside fail2ban for the third experimental configuration. The stronguser account was recreated on this VM with the same credentials. Fail2ban was installed from the EPEL repository and configured with the following SSH jail parameters:

```
[sshd]
enabled  = true
port     = ssh
maxretry = 3
logpath  = %(sshd_log)s
backend  = %(sshd_backend)s
```

This configuration blocked any IP address after three failed authentication attempts.

### 2.2 Reconnaissance

Prior to executing brute-force attacks, the target systems were scanned using Nmap to confirm the availability of the SSH service and identify the exposed port and service version. This step simulated the reconnaissance phase of a real-world attack.

### 2.3 Brute-Force Attack Execution

Attacks were carried out using Hydra, an open-source network login cracker. Hydra was configured to target the SSH service using a password list file (`passwords.txt`). The general command structure was:

```bash
hydra -l <username> -P passwords.txt ssh://<target>:<port>
```

For the fail2ban experiment, the password list was expanded to 80 entries to ensure sufficient attempts would be generated to trigger the rate-limiting mechanism. In a subsequent test, Hydra's thread count was reduced to observe the effect of slower attack rates on fail2ban's detection capability.

### 2.4 Theoretical Analysis

In addition to the practical attacks, a mathematical analysis was performed to estimate the time required for an exhaustive brute-force attack against the strong password. The keyspace was calculated using the formula:

```
combinations = c^n
```

where `c` is the size of the character set and `n` is the password length. Time estimates were derived using:

```
years = c^n / (attempts_per_second × 31,557,600)
```

---

## 3. Results

### 3.1 Reconnaissance Results

Nmap successfully identified the SSH service on the target systems. The Docker container exposed OpenSSH on port 2222, and the Rocky Linux VM exposed OpenSSH on the standard port 22. Both services were confirmed as active and accepting connections.

### 3.2 Brute-Force Attack Results

The results of the brute-force attacks across all three configurations are summarized in the following table:

| Configuration | User       | Password           | Protection | Success | Notes                                                    |
|---------------|------------|--------------------|------------|---------|----------------------------------------------------------|
| Weak          | weakuser   | 123456             | None       | Yes     | Password found almost instantly; ~20 attempts             |
| Strong        | stronguser | Str0ng!Pass#2026   | None       | No      | Password not present in word list; attack exhausted list  |
| Strong + f2b  | stronguser | Str0ng!Pass#2026   | fail2ban   | No      | IP banned after 3 failed attempts; attack terminated      |

In the weak password configuration, Hydra identified the correct password within approximately 20 attempts. The attack completed in negligible time.

In the strong password configuration without protection, Hydra exhausted the entire password list without finding a match. The password `Str0ng!Pass#2026` was not contained in the dictionary file, and therefore the attack failed.

In the strong password configuration with fail2ban, two distinct behaviors were observed. When Hydra operated with its default multi-threaded mode, some threads completed before the ban was applied, but the overall attack was still blocked. When the thread count was reduced to slow the attack rate, fail2ban successfully detected and blocked the attacker before the scan could complete, demonstrating effective rate limiting.

### 3.3 Theoretical Keyspace Analysis

The following table presents the computed keyspace and estimated brute-force duration for a 16-character password across different character sets, assuming average-case discovery (half the keyspace):

| Character Set         | Size (c) | Combinations (c^16)  | Time at 10/s          | Time at 1,000/s       | Time at 1,000,000/s   |
|-----------------------|----------|----------------------|-----------------------|-----------------------|-----------------------|
| Lowercase (a–z)      | 26       | 4.36 × 10^22        | 1.38 × 10^14 years   | 1.38 × 10^12 years   | 1.38 × 10^9 years    |
| Mixed case (a–zA–Z)  | 52       | 2.86 × 10^27        | 9.06 × 10^18 years   | 9.06 × 10^16 years   | 9.06 × 10^13 years   |
| Alphanumeric (a–zA–Z0–9) | 62   | 4.76 × 10^28        | 1.51 × 10^20 years   | 1.51 × 10^18 years   | 1.51 × 10^15 years   |
| Full ASCII (~95)     | 95       | 4.40 × 10^31        | 1.39 × 10^23 years   | 1.39 × 10^21 years   | 1.39 × 10^18 years   |

The password `Str0ng!Pass#2026` utilizes uppercase letters, lowercase letters, digits, and special characters, placing it in the full ASCII category. Even at one million attempts per second, an exhaustive brute-force attack would require approximately 1.39 × 10^18 years — orders of magnitude longer than the estimated age of the universe (1.38 × 10^10 years).

---

## 4. Discussion

### 4.1 Impact of Password Strength

The experimental results clearly demonstrate the critical role of password strength in SSH security. The weak password `123456` — which consistently appears at the top of common password lists such as those published by NordPass and Have I Been Pwned — was compromised almost instantly. This finding aligns with NIST Special Publication 800-63B, which advises against the use of commonly compromised passwords and recommends checking credentials against known breach databases (Grassi et al., 2017).

In contrast, the strong password proved entirely resistant to dictionary-based attacks. The theoretical analysis further confirmed that even pure brute-force approaches are computationally infeasible against passwords of sufficient length and complexity.

### 4.2 Effectiveness of Fail2ban

Fail2ban proved to be a highly effective defensive mechanism against automated brute-force attacks. By monitoring SSH authentication logs and banning IP addresses after three failed attempts, it effectively neutralized Hydra's ability to iterate through the password list. This represents a practical implementation of the defense-in-depth principle, where multiple layers of security controls work together to protect a system.

However, it is important to note that fail2ban is not a complete solution. An attacker could potentially bypass fail2ban through several techniques:

- **Distributed attacks**: Using multiple source IP addresses (e.g., via a botnet or proxy network) to stay below the per-IP threshold.
- **Slow-rate attacks**: Spacing login attempts over long intervals to avoid triggering the ban threshold, though this drastically reduces attack throughput.
- **IP rotation**: Utilizing VPN services or Tor exit nodes to rotate the source address after each ban.

### 4.3 Security vs. Usability Trade-Off

Strong passwords inherently create friction for users. Complex, 16-character passwords are difficult to memorize and prone to being written down or stored insecurely. This trade-off is well-documented in cybersecurity literature and is one of the primary motivations for moving toward key-based SSH authentication, which eliminates password-related vulnerabilities entirely while maintaining usability through SSH agent forwarding and key management tools.

### 4.4 Limitations

Several limitations of this study should be acknowledged. The password list used was relatively small (up to 80 entries), which does not represent the scale of real-world attack dictionaries, which may contain millions or billions of entries. Additionally, Hydra's SSH attack speed is constrained by network latency and the SSH handshake overhead, meaning that the theoretical throughput rates cited in the keyspace analysis are not directly achievable via network-based attacks. Offline attacks against captured password hashes would be significantly faster but require a different attack vector.

---

## 5. Conclusion

This empirical study confirmed three key findings regarding SSH security:

1. **Weak passwords are trivially compromised.** Dictionary-based brute-force attacks can crack common passwords in seconds, underscoring the necessity of enforcing strong password policies.

2. **Strong passwords provide robust protection against brute-force attacks.** A 16-character password using the full printable ASCII character set creates a keyspace that is computationally infeasible to exhaust, even with significant computational resources.

3. **Rate-limiting mechanisms such as fail2ban add a critical defensive layer.** By actively blocking repeated failed authentication attempts, fail2ban can halt automated attacks regardless of password strength.

Based on these findings, the following recommendations are proposed for securing SSH services:

- **Enforce minimum password length and complexity requirements**, consistent with NIST SP 800-63B guidelines.
- **Deploy intrusion prevention systems** such as fail2ban to limit the effectiveness of automated attacks.
- **Transition to key-based authentication** and disable password authentication entirely where feasible.
- **Change the default SSH port** to reduce exposure to automated scanning.
- **Implement multi-factor authentication (MFA)** for an additional layer of security.
- **Regularly audit authentication logs** for signs of brute-force activity.

These measures, applied in combination, embody the defense-in-depth strategy that is fundamental to robust cybersecurity posture.

---

## References

- Grassi, P. A., Garcia, M. E., & Fenton, J. L. (2017). *NIST Special Publication 800-63B: Digital Identity Guidelines — Authentication and Lifecycle Management*. National Institute of Standards and Technology. https://doi.org/10.6028/NIST.SP.800-63b

- Ylonen, T., & Lonvick, C. (2006). *The Secure Shell (SSH) Protocol Architecture*. RFC 4251, Internet Engineering Task Force. https://www.rfc-editor.org/rfc/rfc4251

- van Hauser, T. (2025). *THC Hydra*. https://github.com/vanhauser-thc/thc-hydra

- Lyon, G. (2025). *Nmap: The Network Mapper*. https://nmap.org

- Fail2ban. (2025). *Fail2ban Documentation*. https://www.fail2ban.org

- Ghadeer, A. et al. (2021). Adding Salt to Pepper — A Structured Security Assessment over a Humanoid Robot. *Journal of Information Security and Applications*.

- OpenAI. (2025). *ChatGPT* (Aug 2025) [Large language model]. https://chat.openai.com/
