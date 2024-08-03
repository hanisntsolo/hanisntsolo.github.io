---
layout: post
title: Setting Up Your Codeforces Lamp Real-time Programming Feedback with Docker and IoT
subtitle: How exiting it would be to see your submissions in realtime?
cover-img: assets/img/codeforces/clipfly-ai-20240720142347.jpg
thumbnail-img: assets/img/codeforces/Codeforces-lamp.png
share-img: assets/img/codeforces/clipfly-ai-20240720142347.jpg
tags: [Codeforces, Docker, IoT, Smart Lamp, Competitive Programming, Open Source]
author: Dhirendra Singh
---

<!-- Introduction -->
Welcome to the world of smart coding! 🚀 In this article, we're diving into an exciting project that combines the power of **Codeforces** and **Docker** with IoT to bring real-time feedback to your coding journey. Introducing the **Codeforces Lamp**, a cutting-edge solution designed for developers and competitive programming enthusiasts who want to elevate their coding experience with interactive and visual feedback.

# Setting Up Your Codeforces Lamp: Real-time Programming Feedback with Docker and IoT

## Why Codeforces Lamp?

Have you ever wished for instant feedback on your programming submissions without having to check your screen constantly? Or perhaps you wanted a more engaging way to track your Codeforces ratings? The Codeforces Lamp project is here to make that wish a reality. By integrating with the Codeforces API, this project provides visual feedback through a smart lamp that responds to your profile's rating and submission status.

## How It Works

The Codeforces Lamp project utilizes a smart lamp connected to the internet to display real-time feedback based on your Codeforces activity. Here's how it all comes together:

1. **Integration with Codeforces API**:
   - Fetches real-time data on profile ratings and submissions.
   - Analyzes submission statuses to trigger appropriate lamp responses.

2. **Dockerized Solution**:
   - The entire application runs within a Docker container for easy deployment and scalability.
   - Pre-configured environment ensures all dependencies and configurations are in place.

3. **IoT and Smart Lamp Interaction**:
   - Utilizes Tuya's IoT APIs to communicate with smart lamp devices.
   - Adjusts lamp colors based on coding activities:
     - **In-Process Submissions**: Lamp flickers with a breathing orange effect.
     - **Accepted Submissions**: Lamp turns parrot green as a celebration.
     - **Wrong Submissions**: Lamp displays a red alert.

## Features

The Codeforces Lamp project offers a variety of features designed to enhance your coding experience:

- **Real-Time Feedback**: Instantaneous visual responses to your Codeforces activities.
- **Pre-configured Docker Environment**: Simplifies setup and eliminates the need for manual configurations.
- **Modular Architecture**: Allows for easy extension and customization according to user preferences.
- **Open Source and Community Driven**: Freely available for everyone to use, modify, and contribute to.

## Prerequisites

Before you get started with the Codeforces Lamp project, make sure you have the following:

- **SMARTLIFE App**: Available on [Android](https://play.google.com/store/apps/details?id=com.tuya.smartlife&hl=en_IN) and [iOS](https://apps.apple.com/in/app/smart-life-smart-living/id1115101477).
- **Wipro [Smart Lamp/Light Devices](https://amzn.to/3zKYso2)**: Compatible smart light devices for interaction.
- **Tuya Developer Access**: Required for interacting with Tuya's IoT APIs.
- **Codeforces API Key**: Obtainable from the Codeforces website for API access.

## Setting Up the Codeforces Lamp

Follow these simple steps to set up the Codeforces Lamp and start receiving real-time feedback:

### Step 1: Clone the Repository

Start by cloning the repository from GitHub:

```bash
git clone https://github.com/hanisntsolo/codeforces-lamp.git
cd codeforces-lamp
```
### Step 2: Set Up Environment Variables

```yaml
- CODEFORCES_API_KEY=${CODEFORCES_API_KEY}
- CODEFORCES_API_SECRET=${CODEFORCES_API_SECRET}
- TUYA_ACCESS_ID=${TUYA_ACCESS_ID}
- TUYA_ACCESS_KEY=${TUYA_ACCESS_KEY}
- TUYA_BULB_ID=${TUYA_BULB_ID}
```
### Step 3: Build the Docker Image

```bash
docker build -t codeforces-lamp .
```

### Step 4: Run the Docker Container
```bash
sh rebuild_and_run.sh
```

## Usage
Once everything is set up, the Codeforces Lamp will automatically adjust its colors based on your coding activities. Enjoy the immersive experience of visual coding feedback!

## Contributing
The Codeforces Lamp project is open source and welcomes contributions from the community. If you have ideas for new features, bug fixes, or improvements, please feel free to contribute by creating pull requests or raising issues.

## License
This project is licensed under the MIT License. For more details, see the LICENSE file in the repository.

## Conclusion
The Codeforces Lamp project is more than just a coding tool; it's an innovation that brings coding activities to life. By merging the worlds of competitive programming, Docker, and IoT, this project offers a unique way to interact with your Codeforces profile and submissions.

Whether you're a seasoned programmer or a competitive programming enthusiast, the Codeforces Lamp is designed to enhance your coding journey. Try it out, and let us know your thoughts!

Feel free to connect with me on [LinkedIn](https://www.linkedin.com/in/hanisntsolo/) or email me at ds.pratap1997@gmail.com if you have any questions or suggestions.

Happy coding! 🎉

## Additional Resources

- [Codeforces API Documentation](https://codeforces.com/apiHelp)
- [Docker Official Documentation](https://docs.docker.com/manuals/)
- [Tuya Developer IoT Platform](https://platform.tuya.com/)