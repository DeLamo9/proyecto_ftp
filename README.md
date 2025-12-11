# README.md: Despliegue Automatizado de Servidor FTPS con Autenticación en MariaDB (AWS + Terraform)

# Objetivo y Arquitectura de la Solución

El objetivo es desplegar un servidor FTP que autentica usuarios contra una base de datos externa, cumpliendo con los requisitos de seguridad y aislamiento.

| Componente | Implementación |

| **Instancia FTP** | Servidor **vsftpd** | `main.tf` (`ubuntu_ftp`) y `ftp.sh`. |

| **Instancia BD** | Servidor **MariaDB** | `main.tf` (`ubuntu_bd`) y `mysql.sh`. |

| **Autenticación** | Usuarios virtuales | PAM en `ftp.sh` consulta la BD en `ubuntu_bd`. |

| **Enjaulamiento** | Aislamiento de usuarios | `vsftpd.conf` en `ftp.sh` con `chroot_local_user=YES`. |

| **Modo Pasivo** | Transferencia de datos | Puertos **40000-40100** definidos en firewall y en `vsftpd.conf`. |

| **Firewall** | Reglas de acceso | `aws_security_group` en `main.tf`. |


# Ficheros y Responsabilidades

| Archivo | Tipo | Descripción |

| **`main.tf`** | **Terraform** | Define y orquesta la infraestructura: VPC por defecto, AMIs de Ubuntu 24.04, Key Pair, Security Groups para DB/FTP, dos instancias EC2, IP Elástica (EIP) para el FTP y *Outputs* de las IPs. |

| **`ftp.sh`** | **User Data** | Ejecutado en la Instancia FTP. Instala `vsftpd` y `libpam-mysql`. Utiliza un **Template** para inyectar la IP privada de la BD y la IP pública del FTP. Configura PAM y `vsftpd.conf` para el enjaulamiento y modo pasivo. |

| **`mysql.sh`** | **User Data** | Ejecutado en la Instancia BD. Instala MariaDB, configura `bind-address`, crea la base de datos `vsftpd` y el usuario virtual `David` (`david`). Crea el usuario `ftpuser` para la conexión de vsftpd. |

# Validación de la Conexión FTPS

Utilicé la IP pública elástica (`ftp_public_ip`) mostrada en los Outputs para probar la conexión:

**Cliente:** FTP Cliente (ej: FileZilla).
**Protocolo:** FTPS - FTP sobre SSL/TLS **Explícito**.
**Servidor:** IP pública del Output.
**Puerto:** 21.
**Usuario:** `David`
**Contraseña:** `david`

La conexión fue exitosa, y al navegar, el usuario `David` esta aislado en su carpeta.

# Limpieza de Recursos

```bash
terraform destroy --auto-approve