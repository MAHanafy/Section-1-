# Section-1-
clc;
clear all;
close all;
M=4;
k=log2(M); %Number of bits per symbol
N=k*10^1; % Number of Bits to be processed
nSymbols=N/k; % Total number of symbols
generatedata= randi([0 1],N,1); % Generate bits as 0 & 1
ModulatorPSK = comm.PSKModulator('ModulationOrder',M,'PhaseOffset',0,'BitInput',true); % Define modulator for PSK
DemodulatorPSK = comm.PSKDemodulator('ModulationOrder',M,'PhaseOffset',0,'BitOutput',true);   % Define demodulator for PSK
ModulatorBPSK = comm.BPSKModulator; % Define modulator for BPSK
DemodulatorBPSK = comm.BPSKDemodulator;   % Define demodulator for BPSK
modulatedataBPSK = step(ModulatorBPSK,generatedata);  % Modulate bits using BPSK
modulatedataPSK = step(ModulatorPSK,generatedata);  % Modulate bits using PSK
E_bN_0=-3:10; %Range of Eb/N0 values for Calculation
SNR = E_bN_0 + 10*log10(k); %Conversion into SNR
for j=1:length(SNR) 
    receivedSigPSK(:,j) = awgn(modulatedataPSK,SNR(j),'measured'); %Pass the PSKmodulated data through AWGN for various SNR
    receivedSigBPSK(:,j)= awgn(modulatedataBPSK,E_bN_0(j),'measured'); %Pass the BPSKmodulated data through AWGN for various SNR  
    demodulatedataPSK = step(DemodulatorPSK,receivedSigPSK(:,j)); %Demodulate Noisy PSK Data
    DemodulatedataBPSK = step(DemodulatorBPSK,receivedSigBPSK(:,j)); % Demodulate noisy BPSK data 
    [numerrPSK(j),errPSK(j)] = biterr(generatedata,demodulatedataPSK); %calculate BER of PSK Simulation
    [numerrBPSK(j),errBPSK(j)] = biterr(generatedata,DemodulatedataBPSK); %calculate BER of BPSK Simulation
end
[tberBPSK,serBPSK] = berawgn(E_bN_0,'psk',2,'nondiff'); % Theoretical BER of BPSK in AWGN Channel 
[tberPSK,serPSK] = berawgn(E_bN_0,'psk',M,'nondiff');  % Theoretical BER of PSK in AWGN Channel 
semilogy(E_bN_0,tberBPSK,'ko-','linewidth',2) %Plot Theoretical BER of BPSK in AWGN
hold on
semilogy(E_bN_0,errBPSK,'b-','linewidth',2)  %Plot BER of BPSK Simulated Data
hold on
semilogy(E_bN_0,tberPSK,'g*','linewidth',2) %Plot Theoretical BER of PSK in AWGN
hold on
semilogy(E_bN_0,errPSK,'r-','linewidth',2)  %Plot BER of PSK Simulated Data
axis([-4 12 10^-3 1])
legend ('Theoratical BER BPSK','Simulation BER BPSK','Theoratical BER QPSK','Simulation BER QPSK');
xlabel('E_b/N_0 (dB)')
ylabel('Bit Error Rate')
title('BPSK, QPSK Modulation in AWGN')
grid on
