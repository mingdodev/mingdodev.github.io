---
layout: about
image: /assets/img/post-cover.png
hide_description: true
redirect_from:
  - /download/
---

<head>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
</head>


<style>

    body {
        font-family: 'Inter', sans-serif;
        line-height: 1.6;
        margin: 0;
        padding: 0;
    }

    .name-job-container {
        display: flex;
        justify-content: space-between;
        align-items: flex-start;
        margin-top: 60px;
    }

    .text-container {
        flex: 1;
    }

    .my-name {
        font-weight: 700;
        font-size: 32px;
        color: #333333;
        margin-bottom: 10px;
    }

    .my-job {
        font-weight: 600;
        font-size: 18px;
        color: var(--gray);
        margin-bottom: 20px;
        margin-left: 14px;
    }

    .my-bio {
        font-size: 16px;
        color: #333333;
        line-height: 1.5;
        margin-bottom: 40px;
    }

    .my-about-image {
        width: 20%;
        max-height: 400px;
        object-fit: cover;
        border-radius: 10px;
    }

    .my-section-container {
        display: flex;
        align-items: center;
        margin-bottom: 30px;
    }

    .my-section {
        font-weight: 600;
        font-size: 28px;
        color: #333333;
        text-align: left;
        margin-bottom: 10px;
        margin-right: 20px;
    }

    .my-line {
        height: 1px;
        background-color: #ebebeb;
        flex-grow: 1;
    }

    .my-degree, .my-interest-content {
        font-weight: 400;
        font-size: 16px;
        color: #333333;
        text-align: left;
        margin-left: 10px;
        margin-bottom: 15px;
        line-height: 2;
    }

    .my-experience-content-wrapper {
        display: flex;
        align-items: center;
        margin-bottom: 20px;
    }

    .my-experience-content {
        font-weight: 400;
        font-size: 16px;
        color: #333333;
        text-align: left;
        margin-left: 10px;
        margin-bottom: 0;
    }

    .my-experience-content-detail {
        font-weight: 400;
        font-size: 14px;
        color: var(--gray);
        margin-left: 10px;
        margin-bottom: 0;
    }

    .my-period {
        font-weight: 600;
        font-size: 14px;
        color: #ADB4B9;
        text-align: left;
        margin-left: 10px;
        margin-bottom: 10px;
    }

    .my-skills-wrapper {
        display: flex;
        justify-content: space-around;
        align-items: flex-start;
        gap: 20px; 
        flex-wrap: wrap;
    }

    .my-skills-wrapper > div {
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 10px;
    }

    .my-skills {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 10px;
        padding: 10px;
    }

    .my-skills img {
        height: 40px;
        margin: 10px;
        object-fit: cover;
    }

    .my-skills-opacity {
        opacity: 0.4;
    }

    .my-skill-name {
        text-align: center;
        font-weight: 600;
        margin-bottom: 10px;
        font-size: 16px;
    }

</style>

<div class="resume">
    <!-- 소개 섹션 -->
    <div class="name-job-container">
        <!-- 텍스트 섹션 -->
        <div class="text-container">
            <span class="my-name">Minseo Jang</span>
            <span class="my-job">Server Developer</span>
            <div class="my-bio">
                <br>
                feat: grow <strong>every moment</strong><br>
                style: focus on building <strong>reasonable</strong> solutions<br>
                docs: <strong>share</strong> learned things<br><br>
                Delighted to meet you 👋<br>
                I am a happy developer who loves challenges!
            </div>
        </div>
        <!-- 이미지 섹션 -->
        <img src="/assets/img/profile-img.png" class="my-about-image" alt="My Image">
        <br>
    </div>
    <!-- 경험 섹션 -->
    <div>
        <br>
        <div class="my-section-container">
            <div class="my-section">Experience</div>
            <div class="my-line"></div>
        </div>
        <div class="my-period">March 2024 - Current</div>
        <div class="my-experience-content-wrapper">
            <div class="my-experience-content">AWS Cloud Clubs in South Korea, KHU 2nd Member</div>
        </div>
        <div class="my-period">August - December 2023</div>
        <div class="my-experience-content-wrapper">
            <div class="my-experience-content">Pironeer 1st</div>
            <div class="my-experience-content-detail">Official Development Team for Pirogramming</div>
        </div>
        <div class="my-period">June - August 2023</div>
        <div class="my-experience-content-wrapper">
            <div class="my-experience-content">Pirogramming 19th</div>
            <div class="my-experience-content-detail">University IT Club for Web Development</div>
        </div>
        <br>
    </div>
    <!-- 기술 섹션 -->
    <div>
        <br>
        <div class="my-section-container">
            <div class="my-section">Skills</div>
            <div class="my-line"></div>
        </div>
        <div class="my-skills-wrapper">
            <div>
                <div class="my-skill-name">Programming Languages</div>
                <div class="my-skills">
                    <img src="/assets/img/skills/java.png">
                    <img class="my-skills-opacity" src="/assets/img/skills/python.png">
                </div>
            </div>
            <div>
                <div class="my-skill-name">Backend Development</div>
                <div class="my-skills">
                    <img src="/assets/img/skills/spring.png">
                    <img class="my-skills-opacity" src="/assets/img/skills/expressjs.png">
                    <img class="my-skills-opacity" src="/assets/img/skills/django.png">
                </div>
            </div>
            <div>
                <div class="my-skill-name">DevOps / Cloud Services</div>
                <div class="my-skills">
                    <img src="/assets/img/skills/aws.png">
                    <img src="/assets/img/skills/docker.png">
                </div>
            </div>
        </div>
        <br>
    </div>
    <!-- 최근 관심사 섹션 -->
    <div>
        <br>
        <div class="my-section-container">
            <div class="my-section">Recent Interests</div>
            <div class="my-line"></div>
        </div>
        <ul class="my-interest-content">
        <li> Digging Spring Ecosystem </li>
        <li> How to build a good community for mutual growth </li>
        <li> Seasonal food and Job's Tears Tea </li>
        </ul>
        <br>
    </div>
    <!-- 교육 섹션 -->
    <div>
        <br>
        <div class="my-section-container">
            <div class="my-section">Education</div>
            <div class="my-line"></div>
        </div>
        <div class="my-period">March, 2022 - Current</div>
        <div class="my-degree">Bachelor in <strong>Korean Language</strong> and <strong>Computer Science & Engineering</strong> from Kyung Hee University</div>
        <br>
    </div>
</div>
