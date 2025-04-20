// ==UserScript==
// @name         유튜브 광고 완전 차단 + 자동 스킵
// @namespace    http://tampermonkey.net/
// @version      1.1
// @description  유튜브 광고 및 협찬 광고 자동 차단 (2025년 최신 버전)
// @author       YourName
// @match        *://www.youtube.com/*
// @grant        GM_xmlhttpRequest
// @connect      sponsor.ajay.app
// ==/UserScript==

(function() {
    'use strict';

    console.log("🎉 유튜브 광고 차단 스크립트 실행 중! (2025년 버전)");

    // 🔹 배너 광고 및 오버레이 광고 제거
    function removeAds() {
        // 최신 유튜브 광고 요소 클래스 반영 (2025년 기준)
        let adElements = document.querySelectorAll(
            "ytd-infeed-ad-layout-renderer, .ytp-ad-overlay-slot, .ytp-ad-image-overlay, .ytp-ad-text-overlay"
        );
        adElements.forEach(el => {
            if (el) {
                el.remove();
                console.log("🗑️ 배너/오버레이 광고 제거!");
            }
        });
    }

    // 🔹 동영상 광고 자동 스킵 (프리롤, 미드롤)
    function skipVideoAds() {
        // 최신 스킵 버튼 클래스 반영
        let skipButton = document.querySelector(".ytp-skip-ad-button, .ytp-ad-skip-button-container");
        let video = document.querySelector("video");
        
        if (skipButton) {
            skipButton.click();
            console.log("⏩ 유튜브 광고 스킵!");
        } else if (video && document.querySelector(".ytp-ad-player-overlay")) {
            // 스킵 버튼이 없어도 광고가 재생 중이면 강제로 끝까지 이동
            video.currentTime = video.duration;
            console.log("⏩ 광고 강제 스킵!");
        }
    }

    // 🔹 협찬 광고 자동 스킵 (SponsorBlock API 사용)
    function skipSponsorSegments() {
        let video = document.querySelector("video");
        if (!video) return;

        // 현재 동영상 ID 추출
        let videoId = new URLSearchParams(window.location.search).get("v");
        if (!videoId) return;

        // SponsorBlock API 호출
        GM_xmlhttpRequest({
            method: "GET",
            url: `https://sponsor.ajay.app/api/skipSegments?videoID=${videoId}&categories=["sponsor","selfpromo"]`,
            onload: function(response) {
                if (response.status !== 200) return;

                let segments = JSON.parse(response.responseText);
                if (!segments || segments.length === 0) return;

                let currentTime = video.currentTime;
                segments.forEach(segment => {
                    let start = segment.segment[0];
                    let end = segment.segment[1];

                    if (currentTime >= start && currentTime < end) {
                        video.currentTime = end;
                        console.log(`⏩ 협찬 광고 스킵! (${start}s → ${end}s)`);
                    }
                });
            },
            onerror: function() {
                console.log("⚠️ SponsorBlock API 호출 실패");
            }
        });
    }

    // 🔹 MutationObserver로 DOM 변경 감지 (광고 요소가 동적으로 추가되는 경우 대비)
    function observeDOM() {
        const observer = new MutationObserver((mutations) => {
            mutations.forEach(() => {
                removeAds();
                skipVideoAds();
            });
        });
        observer.observe(document.body, { childList: true, subtree: true });
    }

    // 🔹 500ms마다 광고 확인 (더 빠르게 감지)
    setInterval(() => {
        removeAds();
        skipVideoAds();
        skipSponsorSegments();
    }, 500);

    // 초기 실행 및 DOM 감지 시작
    removeAds();
    skipVideoAds();
    skipSponsorSegments();
    observeDOM();
})();
