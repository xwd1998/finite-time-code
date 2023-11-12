# finite-time-code
Adaptive Event-based Finite-Time H-infinity Filtering for Networked Systems Under Hybrid Attack
clc;
clear all;
close all;
%%仿真参数
h=0.01;
T=30;
N=T/h;
%%系统参数
ms=973;
ks=42720;
cs=3000;
ku=101115;
mu=114;
G0=512*10^(-6);
q0=0.1;
v=12.5;
l=20;
alpha=0.1;
j=1;
count=0;
t(1)=1;
A=[0 0 1 -1;0 0 0 1;-ks/ms 0 -cs/ms cs/ms;ks/mu -ku/mu cs/mu -cs/mu];
B=[0;0;-2*pi*sqrt(G0*v);0];
C=[0 0 0 1];
D=0.1;
L=[0 0 1 0];
W=4;%4
q=[];
x=[];
xf=[];
sigma=0.4;%事件触发条件参数
sigmaM=0.4;
event=0;
sig=0.0005;
del=2;
r=0;
Af=[-0.7365,17.8129,29.2922,-612.7836;1.3187,-34.2351,-3.4164,1062.4031;-0.8749,0.5167,-1.3416,-21.8614;0.9132,-0.3360,-5.1583,-9.4026];
Bf=[1.3131;0.2451;-0.7706;-0.3879];
Cf=[0.4154,-0.0121,-4.2801,-0.3263]; 
%%DoS attack
b_min=2.8;
c_max=2.3;
n=0: h: T;
dos([1:length(n)])=0;
dos1([1:length(n)])=0;
cont([1:length(n)])=0;
M=30;
dos_fre1(1:2:2*M-1)= round (rand(1,M)*(b_min/h));%单区间非攻击长度
dos_fre1(2:2:2*M)= randsrc (1,M, 1:(c_max/h));%单区间攻击长度
dos_fre=cumsum(dos_fre1);%X|H|A&br
dos_end=dos_fre(2:2:length (dos_fre))+1;%AXMAT+1
for m1=1:2:2*M-1
    dos([dos_fre(m1): dos_fre(m1+1)])=1;
    dos_dur(m1)=dos_fre(m1+1)-dos_fre(m1);
end
dos1=dos([1:length(n)-1]);
g_min=3;
h_max=1.5;
f([1:length(n)])=0;
f1([1:length(n)])=0;
cont([1:length(n)])=0;
FF=30;
f_fre1(1:2:2*FF-1)= round (rand(1,FF)*(g_min/h));%单区间非攻击长度
f_fre1(2:2:2*FF)= randsrc (1,FF, 1:(h_max/h));%单区间攻击长度
f_fre=cumsum(f_fre1);%X|H|A&br
f_end=f_fre(2:2:length (f_fre))+1;%AXMAT+1
for ff1=1:2:2*FF-1
    f([f_fre(ff1): f_fre(ff1+1)])=1;
    f_dur(ff1)=f_fre(ff1+1)-f_fre(ff1);
end
f1=f([1:length(n)-1]);
% plot(dos1) DoS信号图
x(:,1)=[0;0;0;0];
xf(:,1)=[0;0;0;0];
% a=rand(3000,1);
for k=1:1:N-1
    if k>=500&&k<=500+l/v*100
        w(k)=alpha*pi*v/l*sin((2*pi*v/l)*k*h);
    else
        w(k)=0;
    end
    if f1(k)==1
        count=count+1;
    else
        count=count;
    end
    x(:,k+1)=(h*A+eye(4))*x(:,k)+h*B*w(k);
    y(k+1)=C*x(:,k+1)+D*w(k);
    z(k+1)=L*x(:,k+1);
    e(k)=y(k+1)-y(t(j));
    lo(k)=-tanh(0.15*y(k));
    xf(:,k+1)=(h*Af+eye(4))*xf(:,k)+(1-f1(k))*h*Bf*y(k+1)+(1-f1(k))*h*Bf*e(k)+f1(k)*lo(k);
    zf(k+1)=Cf*xf(:,k+1);
    qqq=[x;xf];
    www=qqq'*qqq;
    c2=diag(www);
            if sigma<=sigmaM
         sigma =sigma/(1+2*(sigma)*100*(e(k)' *W * e(k)));
        else
            sigma=sigmaM
        end
        sigg1(k)=sigma;
    if dos(k+1)==1
        y(k+1)=0;
        xf(:,k+1)=(h*Af+eye(4))*xf(:,k);
        zf(k+1)=Cf*xf(:,k+1);
        event=event+1;
        sigma=0.4;
%             if e(k)'*e(k)>sig*sig   
%         sigma=sigma*(1/del);  %触发参数变小
%     end
%     if e(k)'*e(k)<sig*sig   
%         if del*sigma<=sigmaM  
%             sigma=sigma*del;  %触发参数变大
%         end
%     end
% sigg1(k)=sigma;
    else if e(k)'*W*e(k)>=sigma*y(k)'*W*y(k)||min(dos_end-k)==0
            j=j+1;
            time(k)=k;
            t(j)=k+1;
            y(k+1)=C*x(:,k+1)+D*w(k);
        else
            time(k)=0;
            y(k+1)=y(t(j));
        end
    end  
            e1(k+1)=z(k+1)-zf(k+1)
        E(k)=e(k)'*W*e(k);
end
for i=1:length(time) %触发次数
    if time(i)>0
        r=r+1;
        trigger(r)=time(i); %触发的时刻1x39维
    end  
end
for i=1:r-1 %间隔次数
    interval(i+1)=trigger(i+1)-trigger(i);%触发间隔大小1x39维
    i=i+1;
end
for i=1:length(trigger)  
trigger_1(i)=trigger(i);
end
for i=1:r-1 
    interval_1(i+1)=trigger_1(i+1)-trigger_1(i); 
    i=i+1;
end
figure(1);%1
plot(dos1)
hold on;
plot(f1)
legend('DoS attack signal','Deception attack signal');
axis([0,3000,-0.5,1.5]);
xticks([0,500,1000,1500,2000,2500,3000]);
xticklabels({'0','5','10','15','20','25','30'})
xticks([0,500,1000,1500,2000,2500,3000]);
xticklabels({'0','5','10','15','20','25','30'})
xlabel('Time(s)');
figure(2);%2
plot(z)
hold on;
plot(zf)
legend('z(t)','z_f(t)');
xticks([0,500,1000,1500,2000,2500,3000]);
xticklabels({'0','5','10','15','20','25','30'})
xlabel('Time(s)');
ylabel('z(t) and its estimation z_f(t)');
figure(3);%3
plot(e1)
legend('e(t)');
xticks([0,500,1000,1500,2000,2500,3000]);
xticklabels({'0','5','10','15','20','25','30'})
xlabel('Time(s)');
ylabel('Filtering error responses');
figure(4);%4
stem(trigger_1,interval_1,'r');%杆状图
xlabel('Time(s)');
ylabel('Release interval and release instant');
xticks([0,500,1000,1500,2000,2500,3000]);
xticklabels({'0','5','10','15','20','25','30'})
figure(5);%5
plot(sigg1);
legend('Adaptive threshold');
axis([0,3000,0,0.5]);
xticks([0,500,1000,1500,2000,2500,3000]);
xticklabels({'0','5','10','15','20','25','30'});
xlabel('Time(s)');
ylabel('Trigger threshold');
figure(6);%6
plot(c2);
legend('c_2 in mechanism 1 responses');
xticks([0,500,1000,1500,2000,2500,3000]);
xticklabels({'0','5','10','15','20','25','30'})
hold on;
xlabel('Time(s)');
ylabel('c_2 responses');
% figure(7);
% plot(dos1);
% hold on;
% plot(e)
% legend('DoS attack signal','e');
% axis([0,3000,-0.05,0.05]);
% xticks([0,500,1000,1500,2000,2500,3000]);
% xticklabels({'0','5','10','15','20','25','30'})
% figure;
% plot(E);
