<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>凌挽拦截 - 防御系统</title>
  <link rel="stylesheet" href="https://cdn.bootcdn.net/ajax/libs/font-awesome/6.4.0/css/all.min.css">
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
      user-select: none;
      -webkit-tap-highlight-color: transparent;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
    }

    body {
      height: 100vh;
      overflow: hidden;
      position: relative;
      background: #000;
      font-family: 'PingFang SC', 'Helvetica Neue', Arial, sans-serif;
      background: radial-gradient(circle at center, #0a0a1a 0%, #000 100%);
      transition: background 0.5s ease;
    }

    body.light-theme {
      background: radial-gradient(circle at center, #e6e9ff 0%, #c9ceff 100%);
      color: #333;
    }

    /* 动态背景效果 */
    body::before {
      content: '';
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: 
        radial-gradient(circle at 20% 50%, rgba(255, 105, 180, 0.1) 0%, transparent 20%),
        radial-gradient(circle at 80% 20%, rgba(0, 122, 255, 0.1) 0%, transparent 20%),
        radial-gradient(circle at 40% 80%, rgba(0, 255, 0, 0.1) 0%, transparent 20%);
      animation: backgroundPulse 10s ease-in-out infinite;
      z-index: 1;
    }

    body.light-theme::before {
      background: 
        radial-gradient(circle at 20% 50%, rgba(255, 105, 180, 0.05) 0%, transparent 20%),
        radial-gradient(circle at 80% 20%, rgba(0, 122, 255, 0.05) 0%, transparent 20%),
        radial-gradient(circle at 40% 80%, rgba(0, 255, 0, 0.05) 0%, transparent 20%);
    }

    @keyframes backgroundPulse {
      0%, 100% { opacity: 0.5; }
      50% { opacity: 0.8; }
    }

    /* 页面容器 */
    .page-container {
      position: relative;
      z-index: 10;
      width: 100%;
      height: 100vh;
      overflow: hidden;
    }

    /* 页面切换 */
    .page {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      opacity: 0;
      visibility: hidden;
      transition: all 0.5s ease;
    }

    .page.active {
      opacity: 1;
      visibility: visible;
    }

    /* 主系统页面 */
    .main-system {
      padding: 20px;
    }

    /* 仪表板页面 */
    .dashboard-system {
      padding: 20px;
      overflow-y: auto;
    }

    /* 主容器 */
    .main-container {
      position: relative;
      z-index: 10;
      display: flex;
      flex-direction: column;
      align-items: center;
      width: 100%;
      max-width: 380px;
      padding: 20px 20px 35px;
      background: rgba(10, 15, 25, 0.92);
      backdrop-filter: blur(15px);
      border-radius: 20px;
      border: 1px solid rgba(255, 255, 255, 0.08);
      box-shadow: 
        0 15px 40px rgba(0, 0, 0, 0.6),
        0 0 0 1px rgba(255, 255, 255, 0.05),
        inset 0 1px 0 rgba(255, 255, 255, 0.1);
      overflow: hidden;
      transition: all 0.3s ease;
    }

    body.light-theme .main-container {
      background: rgba(255, 255, 255, 0.92);
      border: 1px solid rgba(0, 0, 0, 0.08);
      box-shadow: 
        0 15px 40px rgba(0, 0, 0, 0.1),
        0 0 0 1px rgba(0, 0, 0, 0.05),
        inset 0 1px 0 rgba(255, 255, 255, 0.8);
    }

    /* 顶部状态栏 */
    .status-bar {
      width: 100%;
      height: 20px;
      display: flex;
      align-items: center;
      justify-content: center;
      margin-bottom: 15px;
      color: #ff69b4;
      font-size: 13px;
      font-weight: 600;
      letter-spacing: 1px;
    }

    body.light-theme .status-bar {
      color: #007aff;
    }

    /* 头像区域 - 翻转效果 */
    .avatar-container {
      width: 120px;
      height: 120px;
      position: relative;
      margin: 5px 0 20px;
      cursor: pointer;
      perspective: 1000px;
    }

    .avatar-inner {
      position: relative;
      width: 100%;
      height: 100%;
      transition: transform 0.8s cubic-bezier(0.175, 0.885, 0.32, 1.275);
      transform-style: preserve-3d;
    }

    .avatar-container.flipped .avatar-inner {
      transform: rotateY(180deg);
    }

    .avatar-front, .avatar-back {
      position: absolute;
      width: 100%;
      height: 100%;
      backface-visibility: hidden;
      border-radius: 50%;
      overflow: hidden;
    }

    .avatar-back {
      transform: rotateY(180deg);
      background: linear-gradient(135deg, #ff3366, #007aff, #00ff00);
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      color: white;
      text-align: center;
      padding: 15px;
    }

    .avatar-back-content {
      z-index: 2;
      position: relative;
    }

    .avatar-back-title {
      font-size: 16px;
      font-weight: 700;
      margin-bottom: 5px;
      color: white;
      text-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
    }

    .avatar-back-status {
      font-size: 12px;
      color: rgba(255, 255, 255, 0.9);
      text-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
    }

    .avatar-glow {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background: conic-gradient(from 0deg, 
        #ff69b4, #9b59b6, #007aff, #00ff00, #ff69b4
      );
      animation: rotateGlow 4s linear infinite;
      filter: blur(8px);
      opacity: 0.6;
    }

    .avatar-frame {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      border-radius: 50%;
      border: 3px solid transparent;
      background: linear-gradient(45deg, #ff69b4, #007aff, #00ff00) border-box;
      animation: rotateFrame 8s linear infinite;
      padding: 2px;
    }

    .avatar-frame::before {
      content: '';
      position: absolute;
      top: 2px;
      left: 2px;
      right: 2px;
      bottom: 2px;
      background: rgba(20, 25, 40, 0.95);
      border-radius: 50%;
      z-index: 1;
    }

    body.light-theme .avatar-frame::before {
      background: rgba(255, 255, 255, 0.95);
    }

    .avatar-img {
      position: absolute;
      top: 2px;
      left: 2px;
      width: calc(100% - 4px);
      height: calc(100% - 4px);
      border-radius: 50%;
      object-fit: cover;
      z-index: 2;
      transition: transform 0.4s ease;
    }

    .avatar-container:hover .avatar-img {
      transform: scale(1.05);
    }

    .avatar-badge {
      position: absolute;
      bottom: 5px;
      right: 5px;
      width: 24px;
      height: 24px;
      background: linear-gradient(135deg, #ff3366, #ff69b4);
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      color: white;
      font-size: 10px;
      z-index: 3;
      border: 2px solid rgba(255, 255, 255, 0.8);
      animation: badgePulse 2s infinite;
    }

    /* 用户名 */
    .username {
      font-size: 24px;
      font-weight: 700;
      color: #ff69b4;
      letter-spacing: 1px;
      margin: 0 auto 25px;
      text-align: center;
      text-shadow: 0 0 15px rgba(255, 105, 180, 0.5);
      position: relative;
      width: 100%;
      padding: 0 20px;
    }

    body.light-theme .username {
      color: #007aff;
      text-shadow: 0 0 15px rgba(0, 122, 255, 0.3);
    }

    /* 系统状态 */
    .defense-status {
      width: 100%;
      background: rgba(20, 25, 40, 0.8);
      border-radius: 15px;
      padding: 20px;
      margin-bottom: 25px;
      border: 1px solid rgba(255, 255, 255, 0.05);
      transition: all 0.3s ease;
    }

    body.light-theme .defense-status {
      background: rgba(255, 255, 255, 0.8);
      border: 1px solid rgba(0, 0, 0, 0.05);
    }

    .status-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 20px;
      padding-bottom: 15px;
      border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    }

    body.light-theme .status-header {
      border-bottom: 1px solid rgba(0, 0, 0, 0.05);
    }

    .status-title {
      color: #fff;
      font-size: 18px;
      font-weight: 600;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    body.light-theme .status-title {
      color: #333;
    }

    .status-title i {
      color: #ff69b4;
      font-size: 18px;
    }

    body.light-theme .status-title i {
      color: #007aff;
    }

    .status-badge {
      color: #00ff00;
      font-size: 12px;
      padding: 5px 12px;
      background: rgba(0, 255, 0, 0.1);
      border-radius: 12px;
      border: 1px solid rgba(0, 255, 0, 0.2);
      transition: all 0.3s ease;
    }

    .status-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 15px;
    }

    .status-item {
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .status-label {
      color: rgba(255, 255, 255, 0.7);
      font-size: 14px;
      display: flex;
      align-items: center;
      gap: 8px;
    }

    body.light-theme .status-label {
      color: rgba(0, 0, 0, 0.7);
    }

    .status-value {
      color: #fff;
      font-size: 20px;
      font-weight: 700;
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 8px 12px;
      background: rgba(0, 255, 0, 0.05);
      border-radius: 8px;
      border: 1px solid rgba(0, 255, 0, 0.2);
      transition: all 0.3s ease;
    }

    body.light-theme .status-value {
      color: #333;
      background: rgba(0, 255, 0, 0.1);
    }

    /* 按钮区域 */
    .button-container {
      width: 100%;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 15px;
    }

    .button-row {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 12px;
      width: 100%;
    }

    .action-btn {
      width: 56px;
      height: 56px;
      border-radius: 15px;
      border: none;
      background: rgba(30, 35, 50, 0.9);
      color: rgba(255, 255, 255, 0.9);
      font-size: 22px;
      cursor: pointer;
      transition: all 0.3s ease;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      position: relative;
      overflow: hidden;
    }

    body.light-theme .action-btn {
      background: rgba(255, 255, 255, 0.9);
      color: rgba(0, 0, 0, 0.9);
    }

    .action-btn::before {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background: linear-gradient(135deg, rgba(255, 255, 255, 0.1), transparent);
      opacity: 0;
      transition: opacity 0.3s ease;
    }

    .action-btn i {
      margin-bottom: 2px;
      transition: all 0.3s ease;
    }

    .btn-label {
      font-size: 11px;
      color: rgba(255, 255, 255, 0.7);
      margin-top: 2px;
      font-weight: 500;
      transition: color 0.3s ease;
    }

    body.light-theme .btn-label {
      color: rgba(0, 0, 0, 0.7);
    }

    /* 按钮标签 */
    .button-labels {
      display: grid;
      grid-template-columns: repeat(5, 1fr);
      gap: 12px;
      width: 100%;
      margin-top: 5px;
    }

    .button-label {
      text-align: center;
      color: rgba(255, 255, 255, 0.5);
      font-size: 11px;
      font-weight: 500;
    }

    body.light-theme .button-label {
      color: rgba(0, 0, 0, 0.5);
    }

    /* 按钮特效 */
    #backBtn { 
      border: 1px solid rgba(255, 51, 102, 0.3);
    }
    #backBtn:hover { 
      transform: translateY(-5px);
      box-shadow: 0 10px 20px rgba(255, 51, 102, 0.3);
      background: rgba(255, 51, 102, 0.2);
      color: #ff3366;
    }
    #backBtn:hover i { transform: translateX(-3px); }
    #backBtn:hover .btn-label { color: #ff3366; }
    #backBtn:hover::before { opacity: 1; }

    #scanBtn { 
      border: 1px solid rgba(0, 255, 0, 0.3);
    }
    #scanBtn:hover { 
      transform: translateY(-5px);
      box-shadow: 0 10px 20px rgba(0, 255, 0, 0.3);
      background: rgba(0, 255, 0, 0.2);
      color: #00ff00;
    }
    #scanBtn:hover i { animation: scanSpin 0.5s linear; }
    #scanBtn:hover .btn-label { color: #00ff00; }
    #scanBtn:hover::before { opacity: 1; }

    #aiBtn { 
      border: 1px solid rgba(0, 122, 255, 0.3);
    }
    #aiBtn:hover { 
      transform: translateY(-5px);
      box-shadow: 0 10px 20px rgba(0, 122, 255, 0.3);
      background: rgba(0, 122, 255, 0.2);
      color: #007aff;
    }
    #aiBtn:hover i { transform: scale(1.2); }
    #aiBtn:hover .btn-label { color: #007aff; }
    #aiBtn:hover::before { opacity: 1; }

    #menuBtn { 
      border: 1px solid rgba(255, 105, 180, 0.3);
    }
    #menuBtn:hover { 
      transform: translateY(-5px);
      box-shadow: 0 10px 20px rgba(255, 105, 180, 0.3);
      background: rgba(255, 105, 180, 0.2);
      color: #ff69b4;
    }
    #menuBtn:hover i { transform: rotate(90deg); }
    #menuBtn:hover .btn-label { color: #ff69b4; }
    #menuBtn:hover::before { opacity: 1; }

    #homeBtn { 
      border: 1px solid rgba(255, 204, 0, 0.3);
    }
    #homeBtn:hover { 
      transform: translateY(-5px);
      box-shadow: 0 10px 20px rgba(255, 204, 0, 0.3);
      background: rgba(255, 204, 0, 0.2);
      color: #ffcc00;
    }
    #homeBtn:hover i { transform: scale(1.2); }
    #homeBtn:hover .btn-label { color: #ffcc00; }
    #homeBtn:hover::before { opacity: 1; }

    /* 仪表板样式 */
    .dashboard-container {
      position: relative;
      z-index: 10;
      width: 100%;
      height: 100%;
      padding: 20px;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
    }

    .dashboard-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 15px 20px;
      background: rgba(10, 15, 25, 0.9);
      backdrop-filter: blur(10px);
      border-radius: 15px;
      margin-bottom: 20px;
      border: 1px solid rgba(255, 255, 255, 0.08);
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
      transition: all 0.3s ease;
    }

    body.light-theme .dashboard-header {
      background: rgba(255, 255, 255, 0.9);
      border: 1px solid rgba(0, 0, 0, 0.08);
    }

    .header-title {
      display: flex;
      align-items: center;
      gap: 15px;
    }

    .header-title h1 {
      color: #ff69b4;
      font-size: 24px;
      font-weight: 700;
    }

    body.light-theme .header-title h1 {
      color: #007aff;
    }

    .header-title i {
      color: #ff69b4;
      font-size: 28px;
    }

    body.light-theme .header-title i {
      color: #007aff;
    }

    .header-controls {
      display: flex;
      gap: 15px;
    }

    .control-btn {
      background: rgba(255, 255, 255, 0.1);
      border: 1px solid rgba(255, 255, 255, 0.1);
      color: white;
      padding: 10px 20px;
      border-radius: 10px;
      cursor: pointer;
      transition: all 0.3s ease;
      display: flex;
      align-items: center;
      gap: 8px;
      font-weight: 500;
    }

    body.light-theme .control-btn {
      background: rgba(0, 0, 0, 0.05);
      border: 1px solid rgba(0, 0, 0, 0.1);
      color: #333;
    }

    .control-btn:hover {
      background: rgba(255, 105, 180, 0.2);
      transform: translateY(-2px);
      box-shadow: 0 5px 15px rgba(255, 105, 180, 0.2);
    }

    body.light-theme .control-btn:hover {
      background: rgba(0, 122, 255, 0.1);
    }

    .dashboard-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 20px;
      flex: 1;
    }

    .dashboard-card {
      background: rgba(10, 15, 25, 0.9);
      backdrop-filter: blur(10px);
      border-radius: 15px;
      padding: 20px;
      border: 1px solid rgba(255, 255, 255, 0.08);
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
      display: flex;
      flex-direction: column;
      transition: all 0.3s ease;
    }

    body.light-theme .dashboard-card {
      background: rgba(255, 255, 255, 0.9);
      border: 1px solid rgba(0, 0, 0, 0.08);
    }

    .dashboard-card:hover {
      transform: translateY(-5px);
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.4);
    }

    .card-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      padding-bottom: 15px;
      border-bottom: 1px solid rgba(255, 255, 255, 0.05);
    }

    body.light-theme .card-header {
      border-bottom: 1px solid rgba(0, 0, 0, 0.05);
    }

        .card-title {
      color: #ff69b4;
      font-size: 18px;
      font-weight: 600;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    body.light-theme .card-title {
      color: #007aff;
    }

    .card-actions {
      display: flex;
      gap: 10px;
    }

    .card-action {
      background: rgba(255, 255, 255, 0.05);
      border: none;
      color: rgba(255, 255, 255, 0.7);
      width: 30px;
      height: 30px;
      border-radius: 6px;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    body.light-theme .card-action {
      background: rgba(0, 0, 0, 0.05);
      color: rgba(0, 0, 0, 0.7);
    }

    .card-action:hover {
      background: rgba(255, 105, 180, 0.2);
      color: #ff69b4;
      transform: scale(1.1);
    }

    body.light-theme .card-action:hover {
      background: rgba(0, 122, 255, 0.2);
      color: #007aff;
    }

    .system-overview {
      grid-column: span 2;
    }

    .overview-stats {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 15px;
    }

    .stat-item {
      background: rgba(255, 255, 255, 0.05);
      border-radius: 10px;
      padding: 15px;
      display: flex;
      flex-direction: column;
      transition: all 0.3s ease;
    }

    body.light-theme .stat-item {
      background: rgba(0, 0, 0, 0.05);
    }

    .stat-item:hover {
      transform: translateY(-3px);
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
    }

    .stat-label {
      color: rgba(255, 255, 255, 0.7);
      font-size: 14px;
      margin-bottom: 8px;
      display: flex;
      align-items: center;
      gap: 8px;
    }

    body.light-theme .stat-label {
      color: rgba(0, 0, 0, 0.7);
    }

    .stat-value {
      color: white;
      font-size: 24px;
      font-weight: 700;
    }

    body.light-theme .stat-value {
      color: #333;
    }

    .stat-trend {
      font-size: 12px;
      margin-top: 5px;
    }

    .trend-up {
      color: #00ff00;
    }

    .trend-down {
      color: #ff3366;
    }

    .realtime-monitor {
      grid-column: span 3;
      height: 300px;
    }

    .monitor-container {
      flex: 1;
      display: flex;
      flex-direction: column;
    }

    .monitor-graph {
      flex: 1;
      background: rgba(0, 0, 0, 0.2);
      border-radius: 10px;
      position: relative;
      overflow: hidden;
    }

    .graph-line {
      position: absolute;
      bottom: 0;
      width: 100%;
      height: 80%;
      display: flex;
      align-items: flex-end;
      padding: 0 10px;
    }

    .graph-bar {
      flex: 1;
      background: linear-gradient(to top, #ff69b4, #9b59b6);
      margin: 0 2px;
      border-radius: 3px 3px 0 0;
      animation: graphPulse 2s infinite alternate;
      transition: height 0.5s ease;
    }

    @keyframes graphPulse {
      0% { opacity: 0.7; }
      100% { opacity: 1; }
    }

    .graph-labels {
      display: flex;
      justify-content: space-between;
      padding: 10px 5px 0;
      color: rgba(255, 255, 255, 0.5);
      font-size: 12px;
    }

    body.light-theme .graph-labels {
      color: rgba(0, 0, 0, 0.5);
    }

    .threat-map {
      grid-column: span 2;
      height: 300px;
    }

    .map-container {
      flex: 1;
      background: rgba(0, 0, 0, 0.2);
      border-radius: 10px;
      position: relative;
      overflow: hidden;
    }

    .map-point {
      position: absolute;
      width: 12px;
      height: 12px;
      border-radius: 50%;
      background: #ff3366;
      box-shadow: 0 0 10px #ff3366;
      animation: pulse 2s infinite;
      transition: all 0.5s ease;
    }

    @keyframes pulse {
      0% { transform: scale(1); opacity: 1; }
      50% { transform: scale(1.5); opacity: 0.7; }
      100% { transform: scale(1); opacity: 1; }
    }

    .activity-log {
      grid-column: span 1;
      height: 300px;
    }

    .log-container {
      flex: 1;
      overflow-y: auto;
      background: rgba(0, 0, 0, 0.2);
      border-radius: 10px;
      padding: 10px;
    }

    .log-entry {
      padding: 10px;
      border-bottom: 1px solid rgba(255, 255, 255, 0.05);
      display: flex;
      align-items: center;
      gap: 10px;
      transition: all 0.3s ease;
    }

    .log-entry:last-child {
      border-bottom: none;
    }

    .log-entry:hover {
      background: rgba(255, 255, 255, 0.05);
      transform: translateX(5px);
    }

    .log-icon {
      width: 30px;
      height: 30px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 14px;
    }

    .log-icon.threat {
      background: rgba(255, 51, 102, 0.2);
      color: #ff3366;
    }

    .log-icon.security {
      background: rgba(0, 255, 0, 0.2);
      color: #00ff00;
    }

    .log-icon.system {
      background: rgba(0, 122, 255, 0.2);
      color: #007aff;
    }

    .log-content {
      flex: 1;
    }

    .log-message {
      color: white;
      font-size: 14px;
      margin-bottom: 3px;
    }

    body.light-theme .log-message {
      color: #333;
    }

    .log-time {
      color: rgba(255, 255, 255, 0.5);
      font-size: 12px;
    }

    body.light-theme .log-time {
      color: rgba(0, 0, 0, 0.5);
    }

    .performance-metrics {
      grid-column: span 3;
    }

    .metrics-grid {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 15px;
    }

    .metric-item {
      background: rgba(255, 255, 255, 0.05);
      border-radius: 10px;
      padding: 15px;
      text-align: center;
      transition: all 0.3s ease;
    }

    body.light-theme .metric-item {
      background: rgba(0, 0, 0, 0.05);
    }

    .metric-item:hover {
      transform: translateY(-5px);
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
    }

    .metric-value {
      color: white;
      font-size: 28px;
      font-weight: 700;
      margin: 10px 0;
    }

    body.light-theme .metric-value {
      color: #333;
    }

    .metric-label {
      color: rgba(255, 255, 255, 0.7);
      font-size: 14px;
    }

    body.light-theme .metric-label {
      color: rgba(0, 0, 0, 0.7);
    }

    .back-to-main {
      position: fixed;
      bottom: 30px;
      right: 30px;
      background: linear-gradient(135deg, #ff3366, #ff69b4);
      color: white;
      border: none;
      width: 60px;
      height: 60px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
      cursor: pointer;
      box-shadow: 0 5px 15px rgba(255, 51, 102, 0.3);
      z-index: 100;
      transition: all 0.3s ease;
    }

    .back-to-main:hover {
      transform: translateY(-5px) scale(1.1);
      box-shadow: 0 10px 20px rgba(255, 51, 102, 0.4);
    }

    /* 弹窗样式 */
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(0, 0, 0, 0.85);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 1000;
      opacity: 0;
      visibility: hidden;
      transition: all 0.3s ease;
    }

    .modal-overlay.active {
      opacity: 1;
      visibility: visible;
    }

    .modal-content {
      background: rgba(15, 20, 30, 0.95);
      backdrop-filter: blur(20px);
      border-radius: 20px;
      padding: 25px;
      max-width: 400px;
      width: 90%;
      border: 1px solid rgba(255, 255, 255, 0.1);
      box-shadow: 0 20px 50px rgba(0, 0, 0, 0.5);
      transform: translateY(20px) scale(0.9);
      transition: all 0.3s ease;
    }

    body.light-theme .modal-content {
      background: rgba(255, 255, 255, 0.95);
      border: 1px solid rgba(0, 0, 0, 0.1);
    }

    .modal-overlay.active .modal-content {
      transform: translateY(0) scale(1);
    }

    .modal-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 20px;
      padding-bottom: 15px;
      border-bottom: 1px solid rgba(255, 255, 255, 0.1);
    }

    body.light-theme .modal-header {
      border-bottom: 1px solid rgba(0, 0, 0, 0.1);
    }

    .modal-title {
      color: #fff;
      font-size: 20px;
      font-weight: 600;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    body.light-theme .modal-title {
      color: #333;
    }

    .modal-title i {
      color: #ff69b4;
    }

    body.light-theme .modal-title i {
      color: #007aff;
    }

    .close-modal {
      background: none;
      border: none;
      color: rgba(255, 255, 255, 0.5);
      font-size: 24px;
      cursor: pointer;
      transition: color 0.2s ease;
    }

    .close-modal:hover {
      color: #fff;
    }

    body.light-theme .close-modal:hover {
      color: #333;
    }

    .modal-body {
      color: rgba(255, 255, 255, 0.8);
      line-height: 1.6;
      margin-bottom: 25px;
    }

    body.light-theme .modal-body {
      color: rgba(0, 0, 0, 0.8);
    }

    .modal-actions {
      display: flex;
      gap: 10px;
      justify-content: flex-end;
    }

    .modal-btn {
      padding: 10px 20px;
      border: none;
      border-radius: 10px;
      background: rgba(255, 255, 255, 0.1);
      color: #fff;
      cursor: pointer;
      transition: all 0.2s ease;
      font-size: 14px;
      font-weight: 500;
    }

    body.light-theme .modal-btn {
      background: rgba(0, 0, 0, 0.1);
      color: #333;
    }

    .modal-btn.primary {
      background: linear-gradient(135deg, #007aff, #00ff00);
    }

    .modal-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
    }

    /* 扫描进度条 */
    .progress-container {
      margin: 20px 0;
    }

    .progress-info {
      display: flex;
      justify-content: space-between;
      margin-bottom: 8px;
      color: rgba(255, 255, 255, 0.8);
      font-size: 14px;
    }

    body.light-theme .progress-info {
      color: rgba(0, 0, 0, 0.8);
    }

    .progress-bar {
      height: 8px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 4px;
      overflow: hidden;
    }

    body.light-theme .progress-bar {
      background: rgba(0, 0, 0, 0.1);
    }

    .progress-fill {
      height: 100%;
      background: linear-gradient(90deg, #00ff00, #007aff);
      border-radius: 4px;
      width: 0%;
      transition: width 0.3s ease;
    }

    /* 通知系统 */
    .notification {
      position: fixed;
      top: 20px;
      right: 20px;
      background: rgba(20, 25, 40, 0.95);
      backdrop-filter: blur(10px);
      color: white;
      padding: 12px 20px;
      border-radius: 10px;
      border: 1px solid rgba(255, 255, 255, 0.1);
      z-index: 1000;
      transform: translateX(100%);
      opacity: 0;
      transition: all 0.3s ease;
      max-width: 300px;
      font-size: 14px;
      font-weight: 500;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    body.light-theme .notification {
      background: rgba(255, 255, 255, 0.95);
      color: #333;
      border: 1px solid rgba(0, 0, 0, 0.1);
    }

    .notification.show {
      transform: translateX(0);
      opacity: 1;
    }

    .notification.success {
      border-left: 4px solid #00ff00;
      background: rgba(0, 255, 0, 0.1);
    }

    .notification.warning {
      border-left: 4px solid #ffcc00;
      background: rgba(255, 204, 0, 0.1);
    }

    .notification.error {
      border-left: 4px solid #ff3366;
      background: rgba(255, 51, 102, 0.1);
    }

    .notification.info {
      border-left: 4px solid #007aff;
      background: rgba(0, 122, 255, 0.1);
    }

    /* 深度清理相关样式 */
    .deep-clean-btn {
      background: linear-gradient(135deg, #ff3366, #ff69b4);
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 10px;
      font-size: 14px;
      font-weight: 600;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 8px;
      transition: all 0.3s ease;
      margin-top: 10px;
      width: 100%;
      justify-content: center;
    }

    .deep-clean-btn:hover {
      transform: translateY(-2px);
      box-shadow: 0 5px 15px rgba(255, 51, 102, 0.3);
    }

    .deep-clean-btn:disabled {
      background: linear-gradient(135deg, #666, #999);
      cursor: not-allowed;
      transform: none;
      box-shadow: none;
    }

    .deep-clean-btn i {
      animation: cleanSpin 1s linear infinite;
    }

    @keyframes cleanSpin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }

    .clean-stats {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 10px;
      margin-top: 15px;
    }

    .stat-item {
      background: rgba(255, 255, 255, 0.05);
      padding: 10px;
      border-radius: 8px;
      text-align: center;
    }

    .stat-value {
      font-size: 20px;
      font-weight: 700;
      margin-bottom: 5px;
    }

    .stat-label {
      font-size: 12px;
      color: rgba(255, 255, 255, 0.7);
    }

    .clean-result {
      background: rgba(0, 255, 0, 0.1);
      border: 1px solid rgba(0, 255, 0, 0.2);
      border-radius: 10px;
      padding: 15px;
      margin-top: 15px;
      text-align: center;
    }

    .clean-result.error {
      background: rgba(255, 51, 102, 0.1);
      border: 1px solid rgba(255, 51, 102, 0.2);
    }

    .clean-result.warning {
      background: rgba(255, 204, 0, 0.1);
      border: 1px solid rgba(255, 204, 0, 0.2);
    }

    /* 菜单项样式 */
    .menu-item {
      background: rgba(255, 255, 255, 0.05);
      padding: 15px;
      border-radius: 12px;
      text-align: center;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    body.light-theme .menu-item {
      background: rgba(0, 0, 0, 0.05);
    }

    .menu-item:hover {
      transform: translateY(-3px);
      box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
      background: rgba(255, 255, 255, 0.08);
    }

    body.light-theme .menu-item:hover {
      background: rgba(0, 0, 0, 0.08);
    }

    /* 设置面板样式 */
    .settings-panel {
      display: grid;
      gap: 15px;
    }

    .setting-item {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 10px;
      background: rgba(255, 255, 255, 0.05);
      border-radius: 10px;
    }

    body.light-theme .setting-item {
      background: rgba(0, 0, 0, 0.05);
    }

    .setting-label {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .setting-label i {
      color: #ff69b4;
      width: 20px;
    }

    body.light-theme .setting-label i {
      color: #007aff;
    }

    .toggle-switch {
      position: relative;
      width: 50px;
      height: 24px;
    }

    .toggle-switch input {
      opacity: 0;
      width: 0;
      height: 0;
    }

    .toggle-slider {
      position: absolute;
      cursor: pointer;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      background-color: rgba(255, 255, 255, 0.2);
      transition: .4s;
      border-radius: 24px;
    }

    .toggle-slider:before {
      position: absolute;
      content: "";
      height: 18px;
      width: 18px;
      left: 3px;
      bottom: 3px;
      background-color: white;
      transition: .4s;
      border-radius: 50%;
    }

    input:checked + .toggle-slider {
      background-color: #00ff00;
    }

    input:checked + .toggle-slider:before {
      transform: translateX(26px);
    }

    /* 数字计数动画 */
    .count-up {
      animation: countUp 0.6s ease-out;
    }

    @keyframes countUp {
      from { 
        opacity: 0; 
        transform: translateY(10px); 
      }
      to { 
        opacity: 1; 
        transform: translateY(0); 
      }
    }

    /* 动画定义 */
    @keyframes rotateGlow {
      0% {
        transform: rotate(0deg);
        filter: blur(8px) hue-rotate(0deg);
      }
      100% {
        transform: rotate(360deg);
        filter: blur(8px) hue-rotate(360deg);
      }
    }

    @keyframes rotateFrame {
      0% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(360deg);
      }
    }

    @keyframes badgePulse {
      0%, 100% {
        box-shadow: 0 0 0 0 rgba(255, 51, 102, 0.5);
      }
      50% {
        box-shadow: 0 0 0 8px rgba(255, 51, 102, 0);
      }
    }

    @keyframes scanSpin {
      0% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(360deg);
      }
    }

    /* 响应式设计 */
    @media (max-width: 1200px) {
      .dashboard-grid {
        grid-template-columns: repeat(2, 1fr);
      }
      
      .system-overview,
      .threat-map,
      .realtime-monitor,
      .performance-metrics {
        grid-column: span 2;
      }
    }

    @media (max-width: 768px) {
      .dashboard-grid {
        grid-template-columns: 1fr;
      }
      
      .system-overview,
      .threat-map,
      .realtime-monitor,
      .performance-metrics {
        grid-column: span 1;
      }
      
      .overview-stats,
      .metrics-grid {
        grid-template-columns: 1fr;
      }
      
      .header-title h1 {
        font-size: 20px;
      }
    }

    @media (max-width: 400px) {
      .main-container {
        max-width: 95%;
        padding: 15px 15px 30px;
      }
      
      .avatar-container {
        width: 110px;
        height: 110px;
      }
      
      .username {
        font-size: 22px;
      }
      
      .action-btn {
        width: 52px;
        height: 52px;
        font-size: 20px;
      }
      
      .button-labels {
        gap: 10px;
      }
      
      .button-label {
        font-size: 10px;
      }
    }
  </style>
</head>
<body>
  <!-- 页面容器 -->
  <div class="page-container">
    <!-- 主系统页面 -->
    <div class="page main-system active" id="mainPage">
      <div class="main-container">
        <!-- 状态栏 -->
        <div class="status-bar">凌挽拦截防御系统已启动</div>

        <!-- 头像区域 -->
        <div class="avatar-container" id="avatarBox">
          <div class="avatar-inner">
            <div class="avatar-front">
              <div class="avatar-glow"></div>
              <div class="avatar-frame"></div>
              <div class="avatar-badge">
                <i class="fas fa-shield-alt"></i>
              </div>
              <!-- 头像图片由JavaScript动态添加 -->
            </div>
            <div class="avatar-back">
              <div class="avatar-back-content">
                <div class="avatar-back-title">凌挽拦截</div>
                <div class="avatar-back-status">防御系统已激活</div>
              </div>
            </div>
          </div>
        </div>

        <!-- 用户名 -->
        <div class="username">凌挽全火防</div>

                <!-- 系统状态 -->
        <div class="defense-status">
          <div class="status-header">
            <div class="status-title">
              <i class="fas fa-shield-alt"></i>
              系统防御状态
            </div>
            <div class="status-badge" id="systemStatus">在线</div>
          </div>
          <div class="status-grid">
            <div class="status-item">
              <div class="status-label">
                <i class="fas fa-exclamation-triangle"></i>
                入侵检测:
              </div>
              <div class="status-value" id="threatLevel">低风险</div>
            </div>
            <div class="status-item">
              <div class="status-label">
                <i class="fas fa-fire"></i>
                防火墙:
              </div>
              <div class="status-value" id="firewallStatus">高强度</div>
            </div>
            <div class="status-item">
              <div class="status-label">
                <i class="fas fa-network-wired"></i>
                连接数:
              </div>
              <div class="status-value" id="connections">182</div>
            </div>
            <div class="status-item">
              <div class="status-label">
                <i class="fas fa-microchip"></i>
                CPU占用:
              </div>
              <div class="status-value" id="cpuUsage">6%</div>
            </div>
          </div>
        </div>

        <!-- 按钮区域 -->
        <div class="button-container">
          <div class="button-row">
            <button class="action-btn" id="backBtn">
              <i class="fas fa-arrow-left"></i>
              <div class="btn-label">返回</div>
            </button>
            <button class="action-btn" id="scanBtn">
              <i class="fas fa-crosshairs"></i>
              <div class="btn-label">扫描</div>
            </button>
            <button class="action-btn" id="aiBtn">
              <i class="fas fa-robot"></i>
              <div class="btn-label">AI</div>
            </button>
            <button class="action-btn" id="menuBtn">
              <i class="fas fa-bars"></i>
              <div class="btn-label">菜单</div>
            </button>
            <button class="action-btn" id="homeBtn">
              <i class="fas fa-home"></i>
              <div class="btn-label">主页</div>
            </button>
          </div>

          <div class="button-labels">
            <div class="button-label">返回</div>
            <div class="button-label">扫描</div>
            <div class="button-label">AI防御</div>
            <div class="button-label">菜单</div>
            <div class="button-label">主页</div>
          </div>
        </div>
      </div>
    </div>

    <!-- 仪表板页面 -->
    <div class="page dashboard-system" id="dashboardPage">
      <div class="dashboard-container">
        <!-- 头部导航 -->
        <div class="dashboard-header">
          <div class="header-title">
            <i class="fas fa-chart-network"></i>
            <h1>高级监控仪表板 - 凌挽拦截系统</h1>
          </div>
          <div class="header-controls">
            <button class="control-btn" id="refreshDashboard">
              <i class="fas fa-sync-alt"></i>
              刷新数据
            </button>
            <button class="control-btn" id="dashboardSettings">
              <i class="fas fa-cog"></i>
              设置
            </button>
          </div>
        </div>

        <!-- 仪表板网格 -->
        <div class="dashboard-grid">
          <!-- 系统概览 -->
          <div class="dashboard-card system-overview">
            <div class="card-header">
              <div class="card-title">
                <i class="fas fa-tachometer-alt"></i>
                系统概览
              </div>
              <div class="card-actions">
                <button class="card-action">
                  <i class="fas fa-expand"></i>
                </button>
              </div>
            </div>
            <div class="overview-stats">
              <div class="stat-item">
                <div class="stat-label">
                  <i class="fas fa-shield-alt"></i>
                  威胁拦截
                </div>
                <div class="stat-value" id="dashboardThreats">1,284</div>
                <div class="stat-trend trend-up" id="dashboardThreatTrend">+12% 今日</div>
              </div>
              <div class="stat-item">
                <div class="stat-label">
                  <i class="fas fa-network-wired"></i>
                  活跃连接
                </div>
                <div class="stat-value" id="dashboardConnections">342</div>
                <div class="stat-trend trend-up" id="dashboardConnectionsTrend">+8% 今日</div>
              </div>
              <div class="stat-item">
                <div class="stat-label">
                  <i class="fas fa-microchip"></i>
                  CPU 使用率
                </div>
                <div class="stat-value" id="dashboardCpu">24%</div>
                <div class="stat-trend trend-down" id="dashboardCpuTrend">-5% 今日</div>
              </div>
              <div class="stat-item">
                <div class="stat-label">
                  <i class="fas fa-memory"></i>
                  内存使用率
                </div>
                <div class="stat-value" id="dashboardMemory">68%</div>
                <div class="stat-trend trend-up" id="dashboardMemoryTrend">+3% 今日</div>
              </div>
            </div>
          </div>

          <!-- 实时监控 -->
          <div class="dashboard-card realtime-monitor">
            <div class="card-header">
              <div class="card-title">
                <i class="fas fa-wave-square"></i>
                实时流量监控
              </div>
              <div class="card-actions">
                <button class="card-action" id="pauseMonitor">
                  <i class="fas fa-pause"></i>
                </button>
                <button class="card-action">
                  <i class="fas fa-expand"></i>
                </button>
              </div>
            </div>
            <div class="monitor-container">
              <div class="monitor-graph" id="trafficGraph">
                <div class="graph-line" id="graphLine">
                  <!-- 图表柱状图由JavaScript动态生成 -->
                </div>
                <div class="graph-labels">
                  <span>00:00</span>
                  <span>04:00</span>
                  <span>08:00</span>
                  <span>12:00</span>
                  <span>16:00</span>
                  <span>20:00</span>
                  <span>00:00</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 威胁地图 -->
          <div class="dashboard-card threat-map">
            <div class="card-header">
              <div class="card-title">
                <i class="fas fa-globe-americas"></i>
                全球威胁地图
              </div>
              <div class="card-actions">
                <button class="card-action">
                  <i class="fas fa-expand"></i>
                </button>
              </div>
            </div>
            <div class="map-container" id="threatMap">
              <!-- 威胁点由JavaScript动态生成 -->
            </div>
          </div>

          <!-- 活动日志 -->
          <div class="dashboard-card activity-log">
            <div class="card-header">
              <div class="card-title">
                <i class="fas fa-list-alt"></i>
                安全活动日志
              </div>
              <div class="card-actions">
                <button class="card-action" id="filterLogs">
                  <i class="fas fa-filter"></i>
                </button>
              </div>
            </div>
            <div class="log-container" id="dashboardLogs">
              <!-- 日志条目由JavaScript动态生成 -->
            </div>
          </div>

          <!-- 性能指标 -->
          <div class="dashboard-card performance-metrics">
            <div class="card-header">
              <div class="card-title">
                <i class="fas fa-chart-bar"></i>
                性能指标
              </div>
              <div class="card-actions">
                <button class="card-action">
                  <i class="fas fa-expand"></i>
                </button>
              </div>
            </div>
            <div class="metrics-grid">
              <div class="metric-item">
                <i class="fas fa-bolt" style="color: #00ff00; font-size: 24px;"></i>
                <div class="metric-value" id="responseTime">0.8ms</div>
                <div class="metric-label">平均响应时间</div>
              </div>
              <div class="metric-item">
                <i class="fas fa-hdd" style="color: #007aff; font-size: 24px;"></i>
                <div class="metric-value" id="diskHealth">98.7%</div>
                <div class="metric-label">磁盘健康度</div>
              </div>
              <div class="metric-item">
                <i class="fas fa-temperature-high" style="color: #ffcc00; font-size: 24px;"></i>
                <div class="metric-value" id="systemTemp">42°C</div>
                <div class="metric-label">系统温度</div>
              </div>
              <div class="metric-item">
                <i class="fas fa-broadcast-tower" style="color: #ff69b4; font-size: 24px;"></i>
                <div class="metric-value" id="networkRequests">324</div>
                <div class="metric-label">网络请求/分钟</div>
              </div>
            </div>
          </div>
        </div>

        <!-- 返回主系统按钮 -->
        <button class="back-to-main" onclick="switchPage('mainPage')">
          <i class="fas fa-arrow-left"></i>
        </button>
      </div>
    </div>
  </div>

  <!-- 返回弹窗 -->
  <div class="modal-overlay" id="backModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-arrow-left"></i>
          返回确认
        </div>
        <button class="close-modal" data-modal="backModal">&times;</button>
      </div>
      <div class="modal-body">
        <p>确定要返回上一级界面吗？</p>
        <div style="margin-top: 15px; padding: 12px; background: rgba(255, 255, 255, 0.05); border-radius: 10px;">
          <div style="display: flex; align-items: center; margin-bottom: 8px;">
            <i class="fas fa-save" style="color: #007aff; margin-right: 8px;"></i>
            <span>当前设置将自动保存</span>
          </div>
          <div style="display: flex; align-items: center;">
            <i class="fas fa-history" style="color: #00ff00; margin-right: 8px;"></i>
            <span>操作记录将被保留</span>
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="backModal">取消</button>
        <button class="modal-btn primary" id="confirmBack">确认返回</button>
      </div>
    </div>
  </div>

  <!-- 扫描弹窗 -->
  <div class="modal-overlay" id="scanModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-crosshairs"></i>
          系统深度扫描
        </div>
        <button class="close-modal" data-modal="scanModal">&times;</button>
      </div>
      <div class="modal-body">
        <div class="progress-container">
          <div class="progress-info">
            <span>扫描进度</span>
            <span id="scanPercent">0%</span>
          </div>
          <div class="progress-bar">
            <div class="progress-fill" id="scanProgress"></div>
          </div>
        </div>
        <div id="scanResults">
          <div style="color: rgba(255, 255, 255, 0.6); font-size: 14px;">点击开始扫描检测系统威胁...</div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="scanModal">取消</button>
        <button class="modal-btn primary" id="startScan">开始扫描</button>
      </div>
    </div>
  </div>

  <!-- AI防御弹窗 -->
  <div class="modal-overlay" id="aiModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-robot"></i>
          AI智能防御
        </div>
        <button class="close-modal" data-modal="aiModal">&times;</button>
      </div>
      <div class="modal-body">
        <div style="margin-bottom: 20px;">
          <div style="display: flex; justify-content: space-between; margin-bottom: 8px;">
            <span>AI防御状态</span>
            <span id="aiStatus" style="color: #00ff00;">已激活</span>
          </div>
          <div class="progress-bar">
            <div id="aiProgress" class="progress-fill" style="width: 85%; background: linear-gradient(90deg, #007aff, #00ff00);"></div>
          </div>
        </div>
        <div style="background: rgba(255, 255, 255, 0.05); padding: 12px; border-radius: 10px; margin-bottom: 15px;">
          <div style="color: #ff69b4; margin-bottom: 5px;">当前任务</div>
          <div id="aiTask">监控网络流量，分析异常行为</div>
        </div>
        <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 10px;">
          <div style="background: rgba(255, 255, 255, 0.05); padding: 12px; border-radius: 10px; text-align: center;">
            <div style="color: #007aff; font-size: 20px; margin-bottom: 5px;">24</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">威胁拦截</div>
          </div>
          <div style="background: rgba(255, 255, 255, 0.05); padding: 12px; border-radius: 10px; text-align: center;">
            <div style="color: #00ff00; font-size: 20px; margin-bottom: 5px;">98%</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">准确率</div>
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="aiModal">关闭</button>
        <button class="modal-btn primary" id="toggleAI">切换AI模式</button>
      </div>
    </div>
  </div>

  <!-- 菜单弹窗 -->
  <div class="modal-overlay" id="menuModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-bars"></i>
          系统功能菜单
        </div>
        <button class="close-modal" data-modal="menuModal">&times;</button>
      </div>
      <div class="modal-body">
        <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px;">
          <div class="menu-item" data-modal="settingsModal">
            <i class="fas fa-cog" style="color: #ff69b4; font-size: 20px; margin-bottom: 8px;"></i>
            <div style="font-weight: 600; margin-bottom: 5px; font-size: 14px;">系统设置</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">调整系统参数</div>
          </div>
          <div class="menu-item" onclick="switchPage('dashboardPage')">
            <i class="fas fa-chart-network" style="color: #007aff; font-size: 20px; margin-bottom: 8px;"></i>
            <div style="font-weight: 600; margin-bottom: 5px; font-size: 14px;">高级监控</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">详细系统分析</div>
          </div>
          <div class="menu-item" data-modal="securityModal">
            <i class="fas fa-shield-alt" style="color: #00ff00; font-size: 20px; margin-bottom: 8px;"></i>
            <div style="font-weight: 600; margin-bottom: 5px; font-size: 14px;">安全日志</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">查看安全事件</div>
          </div>
          <div class="menu-item" data-modal="performanceModal">
            <i class="fas fa-chart-line" style="color: #ffcc00; font-size: 20px; margin-bottom: 8px;"></i>
            <div style="font-weight: 600; margin-bottom: 5px; font-size: 14px;">性能监控</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">实时监控系统</div>
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="menuModal">关闭</button>
        <button class="modal-btn primary" onclick="showNotification('🚀🚀 快速启动所有功能')">快速启动</button>
      </div>
    </div>
  </div>

  <!-- 主页弹窗 -->
  <div class="modal-overlay" id="homeModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-home"></i>
          系统概览
        </div>
        <button class="close-modal" data-modal="homeModal">&times;</button>
      </div>
      <div class="modal-body">
        <div style="margin-bottom: 20px;">
          <div style="color: #ff69b4; margin-bottom: 12px; display: flex; align-items: center; gap: 8px;">
            <i class="fas fa-info-circle"></i>
            系统信息
          </div>
          <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 12px; margin-bottom: 15px;">
            <div style="background: rgba(255, 255, 255, 0.05); padding: 12px; border-radius: 10px;">
              <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6); margin-bottom: 5px;">在线时间</div>
              <div style="font-size: 18px; font-weight: 600;" id="uptime">00:00:00</div>
            </div>
            <div style="background: rgba(255, 255, 255, 0.05); padding: 12px; border-radius: 10px;">
              <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6); margin-bottom: 5px;">威胁拦截</div>
              <div style="font-size: 18px; font-weight: 600;" id="threatsBlocked">182</div>
            </div>
          </div>
          <div style="background: rgba(255, 255, 255, 0.05); padding: 12px; border-radius: 10px;">
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6); margin-bottom: 5px;">最后扫描时间</div>
            <div style="font-size: 16px; font-weight: 600;" id="lastScan">今天 06:13</div>
          </div>
        </div>
        <div style="color: #00ff00; font-size: 13px; text-align: center; padding: 12px; background: rgba(0, 255, 0, 0.05); border-radius: 10px; border: 1px solid rgba(0, 255, 0, 0.1);">
          <i class="fas fa-check-circle" style="margin-right: 8px;"></i>
          系统运行正常，所有服务已就绪
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="homeModal">关闭</button>
        <button class="modal-btn primary" id="refreshStatus">刷新状态</button>
      </div>
    </div>
  </div>

  <!-- 系统设置弹窗 -->
  <div class="modal-overlay" id="settingsModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-cog"></i>
          系统设置
        </div>
        <button class="close-modal" data-modal="settingsModal">&times;</button>
      </div>
      <div class="modal-body">
        <div class="settings-panel">
          <div class="setting-item">
            <div class="setting-label">
              <i class="fas fa-shield-alt"></i>
              自动防护
            </div>
            <label class="toggle-switch">
              <input type="checkbox" checked>
              <span class="toggle-slider"></span>
            </label>
          </div>
          <div class="setting-item">
            <div class="setting-label">
              <i class="fas fa-bell"></i>
              威胁通知
            </div>
            <label class="toggle-switch">
              <input type="checkbox" checked>
              <span class="toggle-slider"></span>
            </label>
          </div>
          <div class="setting-item">
            <div class="setting-label">
              <i class="fas fa-sync-alt"></i>
              自动更新
            </div>
            <label class="toggle-switch">
              <input type="checkbox">
              <span class="toggle-slider"></span>
            </label>
          </div>
          <div class="setting-item">
            <div class="setting-label">
              <i class="fas fa-gamepad"></i>
              游戏模式
            </div>
            <label class="toggle-switch">
              <input type="checkbox" checked>
              <span class="toggle-slider"></span>
            </label>
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="settingsModal">取消</button>
        <button class="modal-btn primary" onclick="showNotification('✅ 系统设置已保存')">保存设置</button>
      </div>
    </div>
  </div>

  <!-- 网络配置弹窗 -->
  <div class="modal-overlay" id="networkModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-network-wired"></i>
          网络配置
        </div>
        <button class="close-modal" data-modal="networkModal">&times;</button>
      </div>
      <div class="modal-body">
        <div style="margin-bottom: 20px;">
          <div style="color: #007aff; margin-bottom: 10px; font-weight: 600;">当前连接</div>
          <div style="background: rgba(255, 255, 255, 0.05); padding: 15px; border-radius: 10px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 10px;">
              <span>Wi-Fi 连接</span>
              <span style="color: #00ff00;">已连接</span>
            </div>
            <div style="font-size: 14px; color: rgba(255, 255, 255, 0.7);">
              SSID: 凌挽安全网络<br>
              信号强度: 优秀
            </div>
          </div>
        </div>
        <div>
          <div style="color: #ff69b4; margin-bottom: 10px; font-weight: 600;">防火墙规则</div>
          <div style="background: rgba(255, 255, 255, 0.05); padding: 15px; border-radius: 10px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
              <span>入站规则</span>
              <span style="color: #00ff00;">已启用</span>
            </div>
            <div style="display: flex; justify-content: space-between;">
              <span>出站规则</span>
              <span style="color: #00ff00;">已启用</span>
            </div>
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="networkModal">关闭</button>
        <button class="modal-btn primary" onclick="showNotification('🌐🌐 网络配置已应用')">应用配置</button>
      </div>
    </div>
  </div>

   <!-- 安全日志弹窗 -->
  <div class="modal-overlay" id="securityModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-shield-alt"></i>
          安全日志
        </div>
        <button class="close-modal" data-modal="securityModal">&times;</button>
      </div>
      <div class="modal-body">
        <div style="max-height: 300px; overflow-y: auto;">
          <div style="margin-bottom: 15px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
              <span style="color: #ff3366;">威胁拦截</span>
              <span style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">今天 08:30</span>
            </div>
            <div style="font-size: 14px; color: rgba(255, 255, 255, 0.8);">
              已拦截恶意IP: 192.168.1.105 的入侵尝试
            </div>
          </div>
          <div style="margin-bottom: 15px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
              <span style="color: #ffcc00;">可疑活动</span>
              <span style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">今天 07:45</span>
            </div>
            <div style="font-size: 14px; color: rgba(255, 255, 255, 0.8);">
              检测到异常进程行为，已自动隔离
            </div>
          </div>
          <div style="margin-bottom: 15px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
              <span style="color: #00ff00;">系统扫描</span>
              <span style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">今天 06:13</span>
            </div>
            <div style="font-size: 14px; color: rgba(255, 255, 255, 0.8);">
              深度扫描完成，未发现威胁
            </div>
          </div>
          <div style="margin-bottom: 15px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
              <span style="color: #007aff;">防火墙更新</span>
              <span style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">昨天 22:30</span>
            </div>
            <div style="font-size: 14px; color: rgba(255, 255, 255, 0.8);">
              防火墙规则库已更新至最新版本
            </div>
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="securityModal">关闭</button>
        <button class="modal-btn primary" onclick="showNotification('📋📋 安全日志已导出')">导出日志</button>
      </div>
    </div>
  </div>

  <!-- 性能监控弹窗 -->
  <div class="modal-overlay" id="performanceModal">
    <div class="modal-content">
      <div class="modal-header">
        <div class="modal-title">
          <i class="fas fa-chart-line"></i>
          性能监控
        </div>
        <button class="close-modal" data-modal="performanceModal">&times;</button>
      </div>
      <div class="modal-body">
        <div style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; margin-bottom: 20px;">
          <div style="background: rgba(255, 255, 255, 0.05); padding: 15px; border-radius: 10px; text-align: center;">
            <div style="color: #ff3366; font-size: 24px; font-weight: 700; margin-bottom: 5px;">6%</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">CPU使用率</div>
          </div>
          <div style="background: rgba(255, 255, 255, 0.05); padding: 15px; border-radius: 10px; text-align: center;">
            <div style="color: #007aff; font-size: 24px; font-weight: 700; margin-bottom: 5px;">32%</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">内存使用率</div>
          </div>
          <div style="background: rgba(255, 255, 255, 0.05); padding: 15px; border-radius: 10px; text-align: center;">
            <div style="color: #00ff00; font-size: 24px; font-weight: 700; margin-bottom: 5px;">182</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">网络连接数</div>
          </div>
          <div style="background: rgba(255, 255, 255, 0.05); padding: 15px; border-radius: 10px; text-align: center;">
            <div style="color: #ffcc00; font-size: 24px; font-weight: 700; margin-bottom: 5px;">0.2ms</div>
            <div style="font-size: 12px; color: rgba(255, 255, 255, 0.6);">平均响应时间</div>
          </div>
        </div>
        <div style="background: rgba(255, 255, 255, 0.05); padding: 15px; border-radius: 10px;">
          <div style="color: #ff69b4; margin-bottom: 10px; font-weight: 600;">系统状态评估</div>
          <div style="font-size: 14px; color: rgba(255, 255, 255, 0.8);">
            系统运行流畅，性能表现优秀。所有资源使用率均在正常范围内。
          </div>
        </div>
      </div>
      <div class="modal-actions">
        <button class="modal-btn" data-modal="performanceModal">关闭</button>
        <button class="modal-btn primary" onclick="showNotification('📈📈 性能报告已生成')">生成报告</button>
      </div>
    </div>
  </div>

  <!-- 通知系统 -->
  <div class="notification" id="notification">
    <i class="fas fa-info-circle"></i>
    <span id="notificationText">系统已就绪</span>
  </div>

  <script>
    // 系统状态变量
    let systemState = {
      // 主系统状态
      threatLevel: 0, // 0=低风险, 1=中风险, 2=高风险
      connections: 182,
      cpuUsage: 6,
      memoryUsage: 32,
      scanning: false,
      aiActive: true,
      startTime: Date.now(),
      avatarFlipped: false,
      darkTheme: true,
      
      // 清理相关状态
      cleaning: false,
      cleanedThreats: 0,
      systemPerformance: 100,
      lastCleanTime: null,
      threats: 0,
      threatsBlocked: 182,
      
      // 仪表板状态
      monitoringPaused: false,
      currentPage: 'mainPage'
    };

    // 主系统DOM元素
    const avatarBox = document.getElementById('avatarBox');
    const systemStatusElement = document.getElementById('systemStatus');
    const threatLevelElement = document.getElementById('threatLevel');
    const firewallStatusElement = document.getElementById('firewallStatus');
    const connectionsElement = document.getElementById('connections');
    const cpuUsageElement = document.getElementById('cpuUsage');
    
    // 主系统按钮元素
    const backBtn = document.getElementById('backBtn');
    const scanBtn = document.getElementById('scanBtn');
    const aiBtn = document.getElementById('aiBtn');
    const menuBtn = document.getElementById('menuBtn');
    const homeBtn = document.getElementById('homeBtn');
    
    // 主系统弹窗元素
    const backModal = document.getElementById('backModal');
    const scanModal = document.getElementById('scanModal');
    const aiModal = document.getElementById('aiModal');
    const menuModal = document.getElementById('menuModal');
    const homeModal = document.getElementById('homeModal');
    const settingsModal = document.getElementById('settingsModal');
    const networkModal = document.getElementById('networkModal');
    const securityModal = document.getElementById('securityModal');
    const performanceModal = document.getElementById('performanceModal');
    
    // 仪表板DOM元素
    const dashboardPage = document.getElementById('dashboardPage');
    const mainPage = document.getElementById('mainPage');
    const pauseMonitorBtn = document.getElementById('pauseMonitor');
    const refreshDashboardBtn = document.getElementById('refreshDashboard');
    const dashboardLogs = document.getElementById('dashboardLogs');
    
    // 通知元素
    const notification = document.getElementById('notification');
    const notificationText = document.getElementById('notificationText');

    // 初始化头像
    function initAvatar() {
      const avatarImg = document.createElement('img');
      avatarImg.className = 'avatar-img';
      avatarImg.src = 'https://s41.ax1x.com/2025/12/21/pZ32BdI.jpg';
      avatarImg.alt = '凌挽头像';
      avatarImg.onerror = function() {
        this.src = 'data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMTI4IiBoZWlnaHQ9IjEyOCIgdmlld0JveD0iMCAwIDEyOCAxMjgiIGZpbGw9Im5vbmUiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyI+PGNpcmNsZSBjeD0iNjQiIGN5PSI2NCIgcj0iNjIiIGZpbGw9InVybCgjcGFpbnQwX2xpbmVhcl8xNzJfMjU2MikiLz48ZGVmcz48bGluZWFyR3JhZGllbnQgaWQ9InBhaW50MF9saW5Vhcl8xNzJfMjU2MiIgeDE9IjY0IiB5MT0iMiIgeDI9IjY0IiB5Mj0iMTI2IiBncmFkaWVudFVuaXRzPSJ1c2VyU3BhY2VPblVzZSI+PHN0b3Agc3RvcC1jb2xvcj0iI0Y2M0JGNiIvPjxzdG9wIG9mZnNldD0iMSIgc3RvcC1jb2xvcj0iIzAwN0FGRiIvPjwvbGluZWFyR3JhZGllbnQ+PC9kZWZzPjwvc3ZnPg==';
      };
      
      // 添加到头像容器
      const avatarFront = document.querySelector('.avatar-front');
      avatarFront.appendChild(avatarImg);
      
      // 头像点击翻转效果
      avatarBox.addEventListener('click', function() {
        systemState.avatarFlipped = !systemState.avatarFlipped;
        if (systemState.avatarFlipped) {
          this.classList.add('flipped');
          showNotification('🔄 显示系统信息', 'info');
        } else {
          this.classList.remove('flipped');
          showNotification('👤 显示头像', 'info');
        }
        
        // 点击效果
        this.style.transform = 'scale(0.95)';
        setTimeout(() => {
          this.style.transform = '';
        }, 150);
      });
    }

    // 显示通知
    function showNotification(message, type = 'info', duration = 3000) {
      notificationText.textContent = message;
      notification.className = 'notification';
      notification.classList.add(type, 'show');
      
      setTimeout(() => {
        notification.classList.remove('show');
      }, duration);
    }

    // 页面切换
    function switchPage(pageId) {
      // 隐藏所有页面
      document.querySelectorAll('.page').forEach(page => {
        page.classList.remove('active');
      });
      
      // 显示目标页面
      document.getElementById(pageId).classList.add('active');
      systemState.currentPage = pageId;
      
      // 更新通知
      if (pageId === 'dashboardPage') {
        showNotification('🚀 已切换到高级监控仪表板', 'success');
        initDashboard(); // 初始化仪表板
      } else if (pageId === 'mainPage') {
        showNotification('🏠 已返回主系统', 'info');
      }
    }

    // 初始化仪表板
    function initDashboard() {
      // 初始化流量图表
      initTrafficGraph();
      
      // 初始化威胁地图
      initThreatMap();
      
      // 初始化活动日志
      initDashboardLogs();
      
      // 初始化仪表板数据更新
      updateDashboardData();
      
      // 开始数据更新循环
      if (!window.dashboardInterval) {
        window.dashboardInterval = setInterval(updateDashboardData, 3000);
      }
    }

    // 初始化流量图表
    function initTrafficGraph() {
      const graphLine = document.getElementById('graphLine');
      graphLine.innerHTML = '';
      
      for (let i = 0; i < 12; i++) {
        const bar = document.createElement('div');
        bar.className = 'graph-bar';
        const height = Math.floor(Math.random() * 50) + 30;
        bar.style.height = `${height}%`;
        bar.style.animationDelay = `${i * 0.1}s`;
        graphLine.appendChild(bar);
      }
    }

    // 初始化威胁地图
    function initThreatMap() {
      const threatMap = document.getElementById('threatMap');
      threatMap.innerHTML = '';
      
      for (let i = 0; i < 7; i++) {
        const point = document.createElement('div');
        point.className = 'map-point';
        const top = Math.floor(Math.random() * 70) + 10;
        const left = Math.floor(Math.random() * 80) + 10;
        point.style.top = `${top}%`;
        point.style.left = `${left}%`;
        point.style.animationDelay = `${i * 0.2}s`;
        threatMap.appendChild(point);
      }
    }

    // 初始化仪表板日志
    function initDashboardLogs() {
      const logs = [
        {type: 'threat', message: '阻止来自 192.168.1.105 的入侵尝试', time: '2分钟前'},
        {type: 'security', message: '系统扫描完成，未发现威胁', time: '15分钟前'},
        {type: 'system', message: '防火墙规则已更新', time: '1小时前'},
        {type: 'threat', message: '检测到可疑进程 activity_monitor.exe', time: '2小时前'},
        {type: 'security', message: '深度清理完成，移除 3 个威胁', time: '3小时前'}
      ];
      
      dashboardLogs.innerHTML = '';
      logs.forEach(log => addDashboardLog(log.type, log.message, log.time));
    }

    // 添加仪表板日志
    function addDashboardLog(type, message, time) {
      const logEntry = document.createElement('div');
      logEntry.className = 'log-entry';
      
      let iconClass = '';
      let icon = '';
      
      switch(type) {
        case 'threat':
          iconClass = 'threat';
          icon = 'fas fa-shield-virus';
          break;
        case 'security':
          iconClass = 'security';
          icon = 'fas fa-check-circle';
          break;
        case 'system':
          iconClass = 'system';
          icon = 'fas fa-cog';
          break;
      }
      
      logEntry.innerHTML = `
        <div class="log-icon ${iconClass}">
          <i class="${icon}"></i>
        </div>
        <div class="log-content">
          <div class="log-message">${message}</div>
          <div class="log-time">${time}</div>
        </div>
      `;
      
      dashboardLogs.insertBefore(logEntry, dashboardLogs.firstChild);
      
      // 限制日志数量，最多保留10条
      if (dashboardLogs.children.length > 10) {
        dashboardLogs.removeChild(dashboardLogs.lastChild);
      }
      
      // 添加新日志动画
      logEntry.style.opacity = '0';
      logEntry.style.transform = 'translateY(-10px)';
      
      setTimeout(() => {
        logEntry.style.transition = 'all 0.3s ease';
        logEntry.style.opacity = '1';
        logEntry.style.transform = 'translateY(0)';
      }, 10);
    }

    // 更新仪表板数据
    function updateDashboardData() {
      if (systemState.monitoringPaused) return;
      
      // 更新流量图表
      const bars = document.querySelectorAll('.graph-bar');
      bars.forEach(bar => {
        const newHeight = Math.floor(Math.random() * 50) + 30;
        bar.style.height = `${newHeight}%`;
      });
      
      // 更新威胁点位置
      const threatPoints = document.querySelectorAll('.map-point');
      threatPoints.forEach(point => {
        const newTop = Math.floor(Math.random() * 70) + 10;
        const newLeft = Math.floor(Math.random() * 80) + 10;
        point.style.top = `${newTop}%`;
        point.style.left = `${newLeft}%`;
      });
      
      // 更新统计数据
      updateDashboardStats();
      
      // 随机添加日志
      if (Math.random() > 0.7) {
        const logTypes = ['threat', 'security', 'system'];
        const messages = [
          '检测到新的网络连接',
          '系统资源使用正常',
          '防火墙规则已更新',
          '扫描到潜在威胁',
          '性能优化进行中',
          '用户登录成功',
          '数据备份完成'
        ];
        const types = ['threat', 'security', 'system'];
        const randomType = types[Math.floor(Math.random() * types.length)];
        const randomMessage = messages[Math.floor(Math.random() * messages.length)];
        
        addDashboardLog(randomType, randomMessage, '刚刚');
      }
    }

    // 更新仪表板统计
    function updateDashboardStats() {
      // 更新威胁拦截数
      const threatsElement = document.getElementById('dashboardThreats');
      let threats = parseInt(threatsElement.textContent.replace(/,/g, ''));
      threats = Math.max(0, threats + Math.floor(Math.random() * 10) - 2);
      threatsElement.textContent = threats.toLocaleString();
      
      // 更新连接数
      const connectionsElement = document.getElementById('dashboardConnections');
      let connections = parseInt(connectionsElement.textContent.replace(/,/g, ''));
      connections = Math.max(0, connections + Math.floor(Math.random() * 20) - 10);
      connectionsElement.textContent = connections.toLocaleString();
      
      // 更新CPU使用率
      const cpuElement = document.getElementById('dashboardCpu');
      let cpu = parseInt(cpuElement.textContent);
      cpu = Math.max(0, Math.min(100, cpu + Math.floor(Math.random() * 10) - 5));
      cpuElement.textContent = `${cpu}%`;
      
      // 更新内存使用率
      const memoryElement = document.getElementById('dashboardMemory');
      let memory = parseInt(memoryElement.textContent);
      memory = Math.max(0, Math.min(100, memory + Math.floor(Math.random() * 6) - 3));
      memoryElement.textContent = `${memory}%`;
      
      // 更新性能指标
      const responseTimeElement = document.getElementById('responseTime');
      let responseTime = parseFloat(responseTimeElement.textContent);
      responseTime = Math.max(0.1, responseTime + (Math.random() * 0.4 - 0.2));
      responseTimeElement.textContent = `${responseTime.toFixed(1)}ms`;
      
      const diskHealthElement = document.getElementById('diskHealth');
      let diskHealth = parseFloat(diskHealthElement.textContent);
      diskHealth = Math.max(0, Math.min(100, diskHealth + (Math.random() * 2 - 1)));
      diskHealthElement.textContent = `${diskHealth.toFixed(1)}%`;
      
      const systemTempElement = document.getElementById('systemTemp');
      let systemTemp = parseInt(systemTempElement.textContent);
      systemTemp = Math.max(30, Math.min(80, systemTemp + Math.floor(Math.random() * 6) - 3));
      systemTempElement.textContent = `${systemTemp}°C`;
      
      const networkRequestsElement = document.getElementById('networkRequests');
      let networkRequests = parseInt(networkRequestsElement.textContent.replace(/,/g, ''));
      networkRequests = Math.max(0, networkRequests + Math.floor(Math.random() * 50) - 25);
      networkRequestsElement.textContent = networkRequests.toLocaleString();
    }

    // 打开弹窗
    function openModal(modal) {
      modal.classList.add('active');
      document.body.style.overflow = 'hidden';
    }

    // 关闭弹窗
    function closeModal(modal) {
      modal.classList.remove('active');
      document.body.style.overflow = 'auto';
    }

    // 更新防御状态
    function updateDefenseStatus() {
      const threatLevels = ['低风险', '中风险', '高风险'];
      const threatColors = ['#00ff00', '#ffcc00', '#ff3366'];
      const firewallStatuses = ['已启用', '高强度', '极限模式'];
      
      // 随机改变威胁级别（模拟实时变化）
      if (Math.random() > 0.9) {
        systemState.threatLevel = Math.floor(Math.random() * 3);
      }
      
      threatLevelElement.textContent = threatLevels[systemState.threatLevel];
      threatLevelElement.style.color = threatColors[systemState.threatLevel];
      threatLevelElement.style.background = `rgba(${parseInt(threatColors[systemState.threatLevel].slice(1, 3), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(3, 5), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(5, 7), 16)}, 0.1)`;
      threatLevelElement.style.border = `1px solid rgba(${parseInt(threatColors[systemState.threatLevel].slice(1, 3), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(3, 5), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(5, 7), 16)}, 0.2)`;
      
      firewallStatusElement.textContent = firewallStatuses[systemState.threatLevel];
      
      // 更新连接数和CPU占用
      systemState.connections = Math.max(175, Math.min(190, systemState.connections + Math.floor(Math.random() * 6) - 3));
      systemState.cpuUsage = Math.max(4, Math.min(8, systemState.cpuUsage + Math.floor(Math.random() * 4) - 2));
      systemState.memoryUsage = Math.max(28, Math.min(36, systemState.memoryUsage + Math.floor(Math.random() * 4) - 2));
      
      // 添加计数动画
      animateCounter(connectionsElement, systemState.connections);
      animateCounter(cpuUsageElement, `${systemState.cpuUsage}%`);
      
      // 更新系统状态
      systemStatusElement.textContent = systemState.threatLevel === 2 ? '警告' : 
                                       systemState.threatLevel === 1 ? '监控中' : '在线';
      systemStatusElement.style.color = threatColors[systemState.threatLevel];
      systemStatusElement.style.background = `rgba(${parseInt(threatColors[systemState.threatLevel].slice(1, 3), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(3, 5), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(5, 7), 16)}, 0.1)`;
      systemStatusElement.style.border = `1px solid rgba(${parseInt(threatColors[systemState.threatLevel].slice(1, 3), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(3, 5), 16)}, ${parseInt(threatColors[systemState.threatLevel].slice(5, 7), 16)}, 0.2)`;
    }

    // 数字计数动画
    function animateCounter(element, newValue) {
      const oldText = element.textContent;
      const oldValue = parseInt(oldText.replace(/\D/g, '')) || 0;
      const isPercent = oldText.includes('%');
      const newNum = parseInt(newValue.toString().replace(/\D/g, ''));
      
      let start = null;
      const duration = 800;
      
      function step(timestamp) {
        if (!start) start = timestamp;
        const progress = Math.min((timestamp - start) / duration, 1);
        
        // 缓动函数
        const easeProgress = 1 - Math.pow(1 - progress, 3);
        const current = Math.round(oldValue + (newNum - oldValue) * easeProgress);
        
        element.textContent = isPercent ? `${current}%` : current;
        element.classList.add('count-up');
        
        if (progress < 1) {
          requestAnimationFrame(step);
        } else {
          element.textContent = newValue;
          setTimeout(() => element.classList.remove('count-up'), 300);
        }
      }
      
      requestAnimationFrame(step);
    }

    // 扫描功能
    function startScan() {
      if (systemState.scanning) return;
      
      systemState.scanning = true;
      const scanProgress = document.getElementById('scanProgress');
      const scanPercent = document.getElementById('scanPercent');
      const scanResults = document.getElementById('scanResults');
      const startScanBtn = document.getElementById('startScan');
      
      scanProgress.style.width = '0%';
      scanPercent.textContent = '0%';
      scanResults.innerHTML = '<div style="color: rgba(255, 255, 255, 0.6); font-size: 14px;">正在初始化扫描引擎...</div>';
      startScanBtn.disabled = true;
      startScanBtn.textContent = '扫描中...';
      
      let progress = 0;
      const scanInterval = setInterval(() => {
        progress += Math.random() * 20;
        if (progress >= 100) {
          progress = 100;
          clearInterval(scanInterval);
          
          // 扫描完成
          const threats = Math.floor(Math.random() * 5);
          systemState.threats = threats;
          
          if (threats > 0) {
            scanResults.innerHTML = `
              <div style="color: #ffcc00; margin-bottom: 10px;">
                <i class="fas fa-exclamation-triangle"></i> 发现 ${threats} 个潜在威胁
              </div>
              <div style="font-size: 13px; color: rgba(255, 255, 255, 0.6); margin-bottom: 15px;">
                检测到的威胁可能影响系统安全，建议立即深度清理
              </div>
              <div style="background: rgba(255, 204, 0, 0.1); padding: 10px; border-radius: 8px; margin-bottom: 15px;">
                <div style="display: flex; justify-content: space-between; margin-bottom: 5px;">
                  <span>威胁类型</span>
                  <span style="color: #ffcc00;">恶意文件/注册表威胁</span>
                </div>
                <div style="display: flex; justify-content: space-between;">
                  <span>风险级别</span>
                  <span style="color: #ff3366;">${threats > 3 ? '高危' : '中危'}</span>
                </div>
              </div>
              <button id="deepCleanBtn" class="deep-clean-btn" onclick="startDeepClean()">
                <i class="fas fa-shield-virus"></i> 启动深度清理
              </button>
            `;
            
            showNotification(`⚠️ 发现 ${threats} 个威胁项目`, 'warning');
            
            // 更新系统状态
            systemState.threatLevel = Math.min(2, systemState.threatLevel + 1);
            updateDefenseStatus();
          } else {
            scanResults.innerHTML = `
              <div style="color: #00ff00; margin-bottom: 10px;">
                <i class="fas fa-check-circle"></i> 扫描完成！系统安全
              </div>
            `;
            showNotification('✅ 系统安全检查通过', 'success');
          }
          
          startScanBtn.disabled = false;
          startScanBtn.textContent = '重新扫描';
          systemState.scanning = false;
          
          // 更新最后扫描时间
          const now = new Date();
          const lastScan = `今天 ${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
          document.getElementById('lastScan').textContent = lastScan;
          
          // 更新威胁拦截数
          if (threats > 0) {
            const threatsBlocked = document.getElementById('threatsBlocked');
            let current = parseInt(threatsBlocked.textContent);
            animateCounter(threatsBlocked, current + threats);
          }
        } else {
          scanPercent.textContent = `${Math.floor(progress)}%`;
          scanProgress.style.width = `${progress}%`;
          
          // 动态更新扫描状态
          if (progress < 30) {
            scanResults.innerHTML = '<div style="color: rgba(255, 255, 255, 0.6); font-size: 14px;">正在扫描系统文件...</div>';
          } else if (progress < 60) {
            scanResults.innerHTML = '<div style="color: rgba(255, 255, 255, 0.6); font-size: 14px;">正在检查网络连接...</div>';
          } else if (progress < 90) {
            scanResults.innerHTML = '<div style="color: rgba(255, 255, 255, 0.6); font-size: 14px;">正在分析潜在威胁...</div>';
          } else {
            scanResults.innerHTML = '<div style="color: rgba(255, 255, 255, 0.6); font-size: 14px;">正在生成报告...</div>';
          }
        }
      }, 200);
    }

    // 深度清理功能
    function startDeepClean() {
      if (systemState.cleaning) return;
      
      systemState.cleaning = true;
      const deepCleanBtn = document.getElementById('deepCleanBtn');
      const scanResults = document.getElementById('scanResults');
      
      deepCleanBtn.disabled = true;
      deepCleanBtn.innerHTML = '<i class="fas fa-spinner"></i> 深度清理中...';
      
      let cleanProgress = 0;
      const cleanInterval = setInterval(() => {
        cleanProgress += Math.random() * 25;
        if (cleanProgress >= 100) {
          cleanProgress = 100;
          clearInterval(cleanInterval);
          
          // 清理完成
          systemState.cleanedThreats = systemState.threats;
          systemState.threats = 0;
          systemState.threatLevel = 0;
          systemState.systemPerformance = 100;
          systemState.lastCleanTime = new Date();
          
          scanResults.innerHTML = `
            <div style="color: #00ff00; margin-bottom: 10px;">
              <i class="fas fa-check-circle"></i> 深度清理完成！
            </div>
            <div class="clean-result">
              <div style="font-size: 13px; margin-bottom: 10px;">已成功清理 ${systemState.cleanedThreats} 个威胁</div>
              <div class="clean-stats">
                <div class="stat-item">
                  <div class="stat-value" style="color: #00ff00;">${systemState.cleanedThreats}</div>
                  <div class="stat-label">威胁已清理</div>
                </div>
                <div class="stat-item">
                  <div class="stat-value" style="color: #007aff;">100%</div>
                  <div class="stat-label">系统性能</div>
                </div>
              </div>
            </div>
          `;
          
          showNotification(`✅ 成功清理 ${systemState.cleanedThreats} 个威胁`, 'success');
          
          // 更新系统状态
          updateDefenseStatus();
          
          systemState.cleaning = false;
          deepCleanBtn.disabled = false;
          deepCleanBtn.innerHTML = '<i class="fas fa-shield-virus"></i> 启动深度清理';
        } else {
          // 更新清理进度
          const progressText = ['正在隔离威胁文件...', '正在清理注册表...', '正在优化系统...', '正在重启服务...'];
          const index = Math.floor(cleanProgress / 25);
          deepCleanBtn.innerHTML = `<i class="fas fa-spinner"></i> ${progressText[index]} ${Math.floor(cleanProgress)}%`;
        }
      }, 300);
    }

    // AI防御切换
    function toggleAI() {
      systemState.aiActive = !systemState.aiActive;
      const aiStatus = document.getElementById('aiStatus');
      const aiTask = document.getElementById('aiTask');
      const aiProgress = document.getElementById('aiProgress');
      const toggleAI = document.getElementById('toggleAI');
      
      if (systemState.aiActive) {
        aiStatus.textContent = '已激活';
        aiStatus.style.color = '#00ff00';
        aiTask.textContent = '监控网络流量，分析异常行为';
        aiProgress.style.width = '85%';
        aiProgress.style.background = 'linear-gradient(90deg, #007aff, #00ff00)';
        toggleAI.textContent = '关闭AI防御';
        showNotification('🤖 AI智能防御已激活', 'success');
      } else {
        aiStatus.textContent = '已关闭';
        aiStatus.style.color = '#ff3366';
        aiTask.textContent = 'AI防御系统已暂停';
        aiProgress.style.width = '0%';
        toggleAI.textContent = '开启AI防御';
        showNotification('⏸️ AI智能防御已关闭', 'warning');
      }
    }

    // 更新系统信息
    function updateSystemInfo() {
      const now = new Date();
      const uptime = now - systemState.startTime;
      
      // 格式化运行时间
      const hours = Math.floor(uptime / (1000 * 60 * 60));
      const minutes = Math.floor((uptime % (1000 * 60 * 60)) / (1000 * 60));
      const seconds = Math.floor((uptime % (1000 * 60)) / 1000);
      
      const uptimeElement = document.getElementById('uptime');
      uptimeElement.textContent = `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
      
      // 更新威胁拦截数
      const threatsBlocked = document.getElementById('threatsBlocked');
      animateCounter(threatsBlocked, systemState.threatsBlocked);
    }

    // 添加按钮特效
    function addButtonEffects() {
      const buttons = document.querySelectorAll('.action-btn');
      buttons.forEach(button => {
        button.addEventListener('mousedown', function() {
          this.style.transform = 'scale(0.95)';
        });
        
        button.addEventListener('mouseup', function() {
          this.style.transform = '';
        });
        
        button.addEventListener('mouseleave', function() {
          this.style.transform = '';
        });
      });
    }

    // 添加菜单悬停效果
    function addMenuHoverEffects() {
      const menuItems = document.querySelectorAll('.menu-item');
      menuItems.forEach(item => {
        item.addEventListener('mouseenter', function() {
          this.style.transform = 'translateY(-3px)';
          this.style.boxShadow = '0 5px 15px rgba(0, 0, 0, 0.2)';
        });
        
        item.addEventListener('mouseleave', function() {
          this.style.transform = '';
          this.style.boxShadow = '';
        });
      });
    }

    // 初始化键盘快捷键
    function initKeyboardShortcuts() {
      document.addEventListener('keydown', (e) => {
        // 防止在输入框中触发
        if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
        
        // Ctrl+数字键切换页面
        if (e.ctrlKey || e.metaKey) {
          switch(e.key) {
            case '1':
              e.preventDefault();
              switchPage('mainPage');
              break;
            case '2':
              e.preventDefault();
              switchPage('dashboardPage');
              break;
          }
        } else {
          // 功能键
          switch(e.key) {
            case 'Escape':
              e.preventDefault();
              document.querySelectorAll('.modal-overlay.active').forEach(modal => {
                closeModal(modal);
              });
              break;
          }
        }
        
        // 空格键翻转头像
        if (e.key === ' ' && !e.ctrlKey && !e.metaKey && !e.altKey) {
          e.preventDefault();
          avatarBox.click();
        }
      });
    }

    // 主题系统
    function initThemeSystem() {
      // 检查用户偏好
      const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
      systemState.darkTheme = prefersDark;
      
      // 应用初始主题
      applyTheme();
      
      // 在系统设置中添加主题切换
      const themeToggle = document.createElement('div');
      themeToggle.className = 'setting-item';
      themeToggle.innerHTML = `
        <div class="setting-label">
          <i class="fas fa-moon" style="color: #ffcc00;"></i>
          深色主题
        </div>
        <label class="toggle-switch">
          <input type="checkbox" id="themeToggle" ${systemState.darkTheme ? 'checked' : ''}>
          <span class="toggle-slider"></span>
        </label>
      `;
      
      document.querySelector('.settings-panel').appendChild(themeToggle);
      
      // 主题切换事件
      document.getElementById('themeToggle').addEventListener('change', function() {
        systemState.darkTheme = this.checked;
        applyTheme();
        showNotification(
          systemState.darkTheme ? '🌙 已切换到深色主题' : '☀️ 已切换到浅色主题',
          'success'
        );
      });
    }

    function applyTheme() {
      if (systemState.darkTheme) {
        document.body.classList.remove('light-theme');
        document.body.classList.add('dark-theme');
      } else {
        document.body.classList.remove('dark-theme');
        document.body.classList.add('light-theme');
      }
    }

    // 初始化事件监听
    function initEventListeners() {
      // 按钮点击打开弹窗
      backBtn.addEventListener('click', () => openModal(backModal));
      scanBtn.addEventListener('click', () => openModal(scanModal));
      aiBtn.addEventListener('click', () => openModal(aiModal));
      menuBtn.addEventListener('click', () => openModal(menuModal));
      homeBtn.addEventListener('click', () => openModal(homeModal));

      // 菜单项点击打开对应弹窗
      document.querySelectorAll('.menu-item').forEach(item => {
        item.addEventListener('click', (e) => {
          const modalId = e.currentTarget.getAttribute('data-modal');
          if (modalId) {
            const modal = document.getElementById(modalId);
            openModal(modal);
            closeModal(menuModal);
          }
        });
      });

      // 关闭弹窗
      document.querySelectorAll('.close-modal, .modal-btn:not(.primary)').forEach(btn => {
        btn.addEventListener('click', (e) => {
          const modalId = e.target.closest('[data-modal]')?.dataset.modal;
          if (modalId) {
            const modal = document.getElementById(modalId);
            closeModal(modal);
          }
        });
      });

      // 确认返回
      document.getElementById('confirmBack').addEventListener('click', () => {
        closeModal(backModal);
        showNotification('↩️ 已返回上一级界面', 'info');
        // 模拟返回操作
        setTimeout(() => {
          systemState.threatLevel = Math.max(0, systemState.threatLevel - 1);
          updateDefenseStatus();
        }, 500);
      });

      // 开始扫描
      document.getElementById('startScan').addEventListener('click', startScan);

      // 切换AI
      document.getElementById('toggleAI').addEventListener('click', toggleAI);

      // 刷新状态
      document.getElementById('refreshStatus').addEventListener('click', () => {
        updateDefenseStatus();
        updateSystemInfo();
        showNotification('🔄 系统状态已刷新', 'info');
      });

      // 仪表板暂停/继续监控
      pauseMonitorBtn.addEventListener('click', function() {
        systemState.monitoringPaused = !systemState.monitoringPaused;
        
        if (systemState.monitoringPaused) {
          this.innerHTML = '<i class="fas fa-play"></i>';
          this.style.color = '#00ff00';
          addDashboardLog('system', '实时监控已暂停', '刚刚');
        } else {
          this.innerHTML = '<i class="fas fa-pause"></i>';
          this.style.color = '';
          addDashboardLog('system', '实时监控已恢复', '刚刚');
        }
      });

      // 刷新仪表板数据
      refreshDashboardBtn.addEventListener('click', function() {
        // 添加刷新动画
        this.innerHTML = '<i class="fas fa-sync-alt fa-spin"></i> 刷新中...';
        this.disabled = true;
        
        // 模拟数据刷新
        setTimeout(() => {
          updateDashboardData();
          this.innerHTML = '<i class="fas fa-sync-alt"></i> 刷新数据';
          this.disabled = false;
          
          // 显示刷新成功提示
          addDashboardLog('system', '数据已刷新', '刚刚');
        }, 1500);
      });

      // 点击弹窗外部关闭
      document.querySelectorAll('.modal-overlay').forEach(modal => {
        modal.addEventListener('click', (e) => {
          if (e.target === modal) {
            closeModal(modal);
          }
        });
      });

      // 添加按钮特效
      addButtonEffects();
      addMenuHoverEffects();
    }

    // 初始化系统
    function initSystem() {
      initAvatar();
      initThemeSystem();
      updateDefenseStatus();
      updateSystemInfo();
      initEventListeners();
      initKeyboardShortcuts();
      
      // 定期更新状态
      setInterval(updateDefenseStatus, 5000);
      setInterval(updateSystemInfo, 1000);
      setInterval(updateDashboardData, 3000);
      
      // 初始欢迎消息
      setTimeout(() => {
        showNotification('🛡️ 凌挽拦截防御系统已启动', 'success');
      }, 1000);
      
      setTimeout(() => {
        showNotification('⚡ 系统初始化完成，所有功能已就绪', 'info');
      }, 2500);
      
      // 随机系统提示
      setInterval(() => {
        const tips = [
          {msg: '💡 提示：定期扫描保持系统安全', type: 'info'},
          {msg: '⚡ 系统运行流畅，性能优秀', type: 'success'},
          {msg: '🔒 防火墙工作正常，保护您的连接', type: 'info'},
          {msg: '🤖 AI防御系统持续学习中', type: 'success'},
          {msg: '🎮 系统游戏模式已就绪', type: 'info'},
          {msg: '🚀 凌挽拦截，为您的安全护航', type: 'success'}
        ];
        if (Math.random() > 0.7) {
          const tip = tips[Math.floor(Math.random() * tips.length)];
          showNotification(tip.msg, tip.type, 2500);
        }
      }, 8000);
    }

    // 页面加载完成
    window.addEventListener('load', initSystem);
  </script>
</body>
</html>
