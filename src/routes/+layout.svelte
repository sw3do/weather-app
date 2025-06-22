<script>
    import '../app.css';
    import AOS from 'aos';
    import 'aos/dist/aos.css';
    import { onMount } from 'svelte';

    onMount(() => {
        AOS.init({
            duration: 1200,
            once: false,
            offset: 100,
            easing: 'ease-out-cubic',
            delay: 100
        });
    });
</script>

<div class="min-h-screen relative overflow-hidden">
    <!-- Ultra modern gradient background -->
    <div class="fixed inset-0 bg-gradient-to-br from-slate-950 via-violet-950 to-slate-900"></div>
    
    <!-- Animated gradient mesh -->
    <div class="fixed inset-0 opacity-40">
        <div class="gradient-mesh mesh-1"></div>
        <div class="gradient-mesh mesh-2"></div>
        <div class="gradient-mesh mesh-3"></div>
        <div class="gradient-mesh mesh-4"></div>
    </div>
    
    <!-- Floating orbs -->
    <div class="fixed inset-0 overflow-hidden">
        <div class="orb orb-1"></div>
        <div class="orb orb-2"></div>
        <div class="orb orb-3"></div>
        <div class="orb orb-4"></div>
        <div class="orb orb-5"></div>
    </div>
    
    <!-- Enhanced cloud system -->
    <div class="fixed inset-0 overflow-hidden">
        {#each Array(8) as _, i}
            <div class="modern-cloud cloud-{i}"></div>
        {/each}
    </div>
    
    <!-- Twinkling stars -->
    <div class="fixed inset-0">
        {#each Array(50) as _, i}
            <div class="star star-{i}"></div>
        {/each}
    </div>
    
    <!-- Aurora effect -->
    <div class="fixed inset-0 aurora aurora-1"></div>
    <div class="fixed inset-0 aurora aurora-2"></div>
    
    <!-- Content -->
    <div class="relative z-50">
        <slot />
    </div>
</div>

<style>
    /* Gradient mesh background */
    .gradient-mesh {
        position: absolute;
        border-radius: 50%;
        filter: blur(60px);
        animation: meshFloat 20s infinite ease-in-out;
    }
    
    .mesh-1 {
        width: 400px;
        height: 400px;
        background: radial-gradient(circle, rgba(124, 58, 237, 0.6) 0%, transparent 70%);
        top: -200px;
        left: -200px;
        animation-delay: 0s;
    }
    
    .mesh-2 {
        width: 300px;
        height: 300px;
        background: radial-gradient(circle, rgba(236, 72, 153, 0.4) 0%, transparent 70%);
        top: 20%;
        right: -150px;
        animation-delay: -7s;
    }
    
    .mesh-3 {
        width: 500px;
        height: 500px;
        background: radial-gradient(circle, rgba(59, 130, 246, 0.3) 0%, transparent 70%);
        bottom: -250px;
        left: 30%;
        animation-delay: -14s;
    }
    
    .mesh-4 {
        width: 350px;
        height: 350px;
        background: radial-gradient(circle, rgba(168, 85, 247, 0.5) 0%, transparent 70%);
        top: 50%;
        right: 20%;
        animation-delay: -21s;
    }
    
    @keyframes meshFloat {
        0%, 100% { transform: translate(0, 0) rotate(0deg) scale(1); }
        25% { transform: translate(30px, -50px) rotate(90deg) scale(1.1); }
        50% { transform: translate(-20px, 20px) rotate(180deg) scale(0.9); }
        75% { transform: translate(50px, -30px) rotate(270deg) scale(1.05); }
    }
    
    /* Floating orbs */
    .orb {
        position: absolute;
        border-radius: 50%;
        filter: blur(1px);
        animation: orbFloat 15s infinite ease-in-out;
    }
    
    .orb-1 {
        width: 60px;
        height: 60px;
        background: linear-gradient(45deg, rgba(124, 58, 237, 0.3), rgba(236, 72, 153, 0.3));
        top: 10%;
        left: 10%;
        animation-delay: 0s;
    }
    
    .orb-2 {
        width: 40px;
        height: 40px;
        background: linear-gradient(45deg, rgba(59, 130, 246, 0.4), rgba(147, 197, 253, 0.4));
        top: 60%;
        right: 15%;
        animation-delay: -5s;
    }
    
    .orb-3 {
        width: 80px;
        height: 80px;
        background: linear-gradient(45deg, rgba(168, 85, 247, 0.2), rgba(196, 181, 253, 0.2));
        bottom: 20%;
        left: 20%;
        animation-delay: -10s;
    }
    
    .orb-4 {
        width: 35px;
        height: 35px;
        background: linear-gradient(45deg, rgba(236, 72, 153, 0.5), rgba(251, 207, 232, 0.5));
        top: 30%;
        left: 70%;
        animation-delay: -15s;
    }
    
    .orb-5 {
        width: 50px;
        height: 50px;
        background: linear-gradient(45deg, rgba(34, 197, 94, 0.3), rgba(134, 239, 172, 0.3));
        bottom: 40%;
        right: 30%;
        animation-delay: -20s;
    }
    
    @keyframes orbFloat {
        0%, 100% { transform: translateY(0px) translateX(0px) scale(1); opacity: 0.3; }
        25% { transform: translateY(-40px) translateX(20px) scale(1.1); opacity: 0.6; }
        50% { transform: translateY(-80px) translateX(-10px) scale(0.9); opacity: 0.4; }
        75% { transform: translateY(-40px) translateX(30px) scale(1.05); opacity: 0.7; }
    }
    
    /* Modern cloud system */
    .modern-cloud {
        position: absolute;
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        backdrop-filter: blur(10px);
        border-radius: 100px;
        animation: modernCloudFloat 30s infinite linear;
    }
    
    .modern-cloud::before,
    .modern-cloud::after {
        content: '';
        position: absolute;
        background: rgba(255, 255, 255, 0.05);
        border: 1px solid rgba(255, 255, 255, 0.1);
        backdrop-filter: blur(10px);
        border-radius: 50%;
    }
    
    .cloud-0 {
        width: 120px;
        height: 50px;
        top: 15%;
        left: -120px;
        animation-duration: 35s;
    }
    
    .cloud-0::before {
        width: 60px;
        height: 60px;
        top: -30px;
        left: 20px;
    }
    
    .cloud-0::after {
        width: 70px;
        height: 50px;
        top: -20px;
        right: 20px;
    }
    
    .cloud-1 {
        width: 100px;
        height: 40px;
        top: 25%;
        left: -100px;
        animation-duration: 40s;
        animation-delay: -5s;
    }
    
    .cloud-1::before {
        width: 50px;
        height: 50px;
        top: -25px;
        left: 15px;
    }
    
    .cloud-1::after {
        width: 60px;
        height: 40px;
        top: -15px;
        right: 15px;
    }
    
    .cloud-2 {
        width: 90px;
        height: 35px;
        top: 45%;
        left: -90px;
        animation-duration: 45s;
        animation-delay: -10s;
    }
    
    .cloud-2::before {
        width: 45px;
        height: 45px;
        top: -22px;
        left: 12px;
    }
    
    .cloud-2::after {
        width: 55px;
        height: 35px;
        top: -12px;
        right: 12px;
    }
    
    .cloud-3 {
        width: 110px;
        height: 45px;
        top: 65%;
        left: -110px;
        animation-duration: 38s;
        animation-delay: -15s;
    }
    
    .cloud-3::before {
        width: 55px;
        height: 55px;
        top: -27px;
        left: 18px;
    }
    
    .cloud-3::after {
        width: 65px;
        height: 45px;
        top: -17px;
        right: 18px;
    }
    
    .cloud-4 {
        width: 80px;
        height: 30px;
        top: 75%;
        left: -80px;
        animation-duration: 42s;
        animation-delay: -20s;
    }
    
    .cloud-4::before {
        width: 40px;
        height: 40px;
        top: -20px;
        left: 10px;
    }
    
    .cloud-4::after {
        width: 50px;
        height: 30px;
        top: -10px;
        right: 10px;
    }
    
    .cloud-5 {
        width: 95px;
        height: 38px;
        top: 5%;
        left: -95px;
        animation-duration: 48s;
        animation-delay: -25s;
    }
    
    .cloud-5::before {
        width: 47px;
        height: 47px;
        top: -23px;
        left: 14px;
    }
    
    .cloud-5::after {
        width: 57px;
        height: 38px;
        top: -13px;
        right: 14px;
    }
    
    .cloud-6 {
        width: 85px;
        height: 32px;
        top: 35%;
        left: -85px;
        animation-duration: 36s;
        animation-delay: -30s;
    }
    
    .cloud-6::before {
        width: 42px;
        height: 42px;
        top: -21px;
        left: 11px;
    }
    
    .cloud-6::after {
        width: 52px;
        height: 32px;
        top: -11px;
        right: 11px;
    }
    
    .cloud-7 {
        width: 105px;
        height: 42px;
        top: 55%;
        left: -105px;
        animation-duration: 44s;
        animation-delay: -35s;
    }
    
    .cloud-7::before {
        width: 52px;
        height: 52px;
        top: -26px;
        left: 16px;
    }
    
    .cloud-7::after {
        width: 62px;
        height: 42px;
        top: -16px;
        right: 16px;
    }
    
    @keyframes modernCloudFloat {
        0% { transform: translateX(-200px); }
        100% { transform: translateX(calc(100vw + 200px)); }
    }
    
    /* Twinkling stars */
    .star {
        position: absolute;
        width: 2px;
        height: 2px;
        background: white;
        border-radius: 50%;
        animation: twinkle 3s infinite;
    }
    
    .star-0 { top: 5%; left: 5%; animation-delay: 0s; }
    .star-1 { top: 10%; left: 15%; animation-delay: 0.2s; }
    .star-2 { top: 15%; left: 25%; animation-delay: 0.4s; }
    .star-3 { top: 20%; left: 35%; animation-delay: 0.6s; }
    .star-4 { top: 25%; left: 45%; animation-delay: 0.8s; }
    .star-5 { top: 30%; left: 55%; animation-delay: 1s; }
    .star-6 { top: 35%; left: 65%; animation-delay: 1.2s; }
    .star-7 { top: 40%; left: 75%; animation-delay: 1.4s; }
    .star-8 { top: 45%; left: 85%; animation-delay: 1.6s; }
    .star-9 { top: 50%; left: 95%; animation-delay: 1.8s; }
    .star-10 { top: 8%; left: 80%; animation-delay: 2s; }
    .star-11 { top: 18%; left: 90%; animation-delay: 2.2s; }
    .star-12 { top: 28%; left: 10%; animation-delay: 2.4s; }
    .star-13 { top: 38%; left: 20%; animation-delay: 2.6s; }
    .star-14 { top: 48%; left: 30%; animation-delay: 2.8s; }
    .star-15 { top: 58%; left: 40%; animation-delay: 3s; }
    .star-16 { top: 68%; left: 50%; animation-delay: 0.3s; }
    .star-17 { top: 78%; left: 60%; animation-delay: 0.7s; }
    .star-18 { top: 88%; left: 70%; animation-delay: 1.1s; }
    .star-19 { top: 3%; left: 40%; animation-delay: 1.5s; }
    .star-20 { top: 13%; left: 50%; animation-delay: 1.9s; }
    .star-21 { top: 23%; left: 60%; animation-delay: 2.3s; }
    .star-22 { top: 33%; left: 70%; animation-delay: 2.7s; }
    .star-23 { top: 43%; left: 80%; animation-delay: 0.1s; }
    .star-24 { top: 53%; left: 90%; animation-delay: 0.5s; }
    .star-25 { top: 63%; left: 5%; animation-delay: 0.9s; }
    .star-26 { top: 73%; left: 15%; animation-delay: 1.3s; }
    .star-27 { top: 83%; left: 25%; animation-delay: 1.7s; }
    .star-28 { top: 93%; left: 35%; animation-delay: 2.1s; }
    .star-29 { top: 7%; left: 45%; animation-delay: 2.5s; }
    .star-30 { top: 17%; left: 55%; animation-delay: 2.9s; }
    .star-31 { top: 27%; left: 65%; animation-delay: 0.4s; }
    .star-32 { top: 37%; left: 75%; animation-delay: 0.8s; }
    .star-33 { top: 47%; left: 85%; animation-delay: 1.2s; }
    .star-34 { top: 57%; left: 95%; animation-delay: 1.6s; }
    .star-35 { top: 67%; left: 12%; animation-delay: 2s; }
    .star-36 { top: 77%; left: 22%; animation-delay: 2.4s; }
    .star-37 { top: 87%; left: 32%; animation-delay: 2.8s; }
    .star-38 { top: 12%; left: 42%; animation-delay: 0.2s; }
    .star-39 { top: 22%; left: 52%; animation-delay: 0.6s; }
    .star-40 { top: 32%; left: 62%; animation-delay: 1s; }
    .star-41 { top: 42%; left: 72%; animation-delay: 1.4s; }
    .star-42 { top: 52%; left: 82%; animation-delay: 1.8s; }
    .star-43 { top: 62%; left: 92%; animation-delay: 2.2s; }
    .star-44 { top: 72%; left: 8%; animation-delay: 2.6s; }
    .star-45 { top: 82%; left: 18%; animation-delay: 3s; }
    .star-46 { top: 92%; left: 28%; animation-delay: 0.3s; }
    .star-47 { top: 6%; left: 38%; animation-delay: 0.7s; }
    .star-48 { top: 16%; left: 48%; animation-delay: 1.1s; }
    .star-49 { top: 26%; left: 58%; animation-delay: 1.5s; }
    
    @keyframes twinkle {
        0%, 100% { opacity: 0.2; transform: scale(1); }
        50% { opacity: 1; transform: scale(1.5); }
    }
    
    /* Aurora effect */
    .aurora {
        position: absolute;
        width: 100%;
        height: 100%;
        opacity: 0.1;
        animation: aurora 25s infinite ease-in-out;
    }
    
    .aurora-1 {
        background: linear-gradient(90deg, 
            transparent 0%, 
            rgba(34, 197, 94, 0.3) 25%, 
            rgba(59, 130, 246, 0.3) 50%, 
            rgba(168, 85, 247, 0.3) 75%, 
            transparent 100%);
        animation-delay: 0s;
    }
    
    .aurora-2 {
        background: linear-gradient(45deg, 
            transparent 0%, 
            rgba(236, 72, 153, 0.2) 30%, 
            rgba(124, 58, 237, 0.2) 70%, 
            transparent 100%);
        animation-delay: -12s;
    }
    
    @keyframes aurora {
        0%, 100% { 
            transform: translateX(-100%) rotate(0deg); 
            opacity: 0.1; 
        }
        50% { 
            transform: translateX(100%) rotate(180deg); 
            opacity: 0.3; 
        }
    }
</style> 