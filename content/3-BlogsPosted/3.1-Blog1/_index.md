---
title: "Blog 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Custom OS installation now available on AWS DeepRacer devices

While exploring recent AWS Blog articles, I found an interesting update about **AWS DeepRacer**.

Previously, I mainly knew AWS DeepRacer as a 1/18-scale autonomous race car used to learn Machine Learning and Reinforcement Learning. Developers can train models in a cloud environment and deploy them to the physical vehicle for testing.

After reading this article, I learned that DeepRacer devices originally booted AWS-signed operating systems, including Ubuntu 16.04 and Ubuntu 20.04. As these operating system versions are no longer supported, continuing experimentation and installing modern software has become more difficult.

Link blog: https://www.facebook.com/groups/awsstudygroupfcj/posts/2220039238761036

## Developer Bootloader

The most interesting update is the release of the **Developer Bootloader**, which allows developers to install custom operating systems or third-party Linux distributions on AWS DeepRacer devices.

The Developer Bootloader is based on the open-source `shim` project. When the built-in signature verification fails, it checks developer certificates stored in:

```text
/EFI/DEVELOPER/certs/
```

Developers can manage their own certificates using standard tools such as OpenSSL and `sbsign` while maintaining cryptographic verification during the boot process.

## Developer Mode Indicators

When developer certificates are being used, the device provides several clear indicators:

- A Developer Mode warning appears through an HDMI display.
- The device lights blink “DEVELOPER MODE” in Morse code.
- A boot delay gives the user time to recognize that Developer Mode is active.

I think this is a useful design because developers gain more control over the device while remaining aware of its current security state.

## Installation Options

AWS introduces three main ways to use the Developer Bootloader.

### 1. Install a Community Distribution

The AWS DeepRacer community has created a distribution based on Ubuntu 24.04 and ROS2 Jazzy. The Developer Bootloader is already included, allowing users to get started without manually configuring the entire Linux boot process.

### 2. Install a Third-Party Linux Distribution

Developers can install Ubuntu or another Linux distribution, replace the `BOOTX64.EFI` file with the Developer Shim, and add the required certificates.

This option is suitable for users who want more control over the operating system version, software packages, hardware drivers, and development tools.

### 3. Build a Custom Operating System

Advanced developers can also build their own operating system for AWS DeepRacer.

The Developer Shim is placed at:

```text
EFI/BOOT/BOOTX64.EFI
```

The operating system kernel or bootloader is placed at:

```text
EFI/BOOT/GRUBX64.EFI
```

The public certificate is stored under the `DEVELOPER/certs` directory for verification during startup.

## What I Learned

After researching this update, I realized that the new bootloader provides much more than the ability to install a newer Ubuntu version.

Developers can now:

- Install modern Linux operating systems.
- Experiment with newer ROS2 packages.
- Add custom hardware drivers.
- Develop new vehicle-control algorithms.
- Build Machine Learning and Robotics projects.
- Extend the useful life of existing DeepRacer hardware.

The process is also reversible. Users can restore their devices to the original AWS configuration when necessary.

## Conclusion

This article helped me understand how AWS is expanding the development capabilities of AWS DeepRacer through the new Developer Bootloader.

Instead of using DeepRacer only for Reinforcement Learning and racing, developers can now control the operating system, software stack, and development environment of the device.

In my opinion, this update is valuable to the AWS DeepRacer community because it extends the lifecycle of the hardware and creates more opportunities for Machine Learning, ROS2, and Robotics experimentation.

<p align="center">
  <strong>Reference Article:</strong>
  <a href="https://aws.amazon.com/blogs/machine-learning/custom-os-installation-now-available-on-aws-deepracer-devices/" target="_blank">
    AWS Artificial Intelligence Blog
  </a>
</p>