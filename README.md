# BMEG-321


% Author: Scott
% Date: March 11th

%% Data Loading
addpath("Matlab/BMEG 321/Lab 6/");
normal = readtable("Normal Data Filt");
cuff = readtable("Cuff Data Filt");
spi = readtable("spirometer.csv");

head(normal)
head(spi)


%% Filter ECG Signal (Bandpass: 0.5 - 50 Hz)
Fs = 1000; % Sampling frequency
low_cutoff = 0.5;
high_cutoff = 50;

[b, a] = butter(2, [low_cutoff high_cutoff] / (Fs / 2), 'bandpass');

% Filter all ECG leads (choose later which lead is best)
nor_ECG1_filt = filtfilt(b, a, normal.ECG1);
nor_ECG2_filt = filtfilt(b, a, normal.ECG2);
nor_ECG3_filt = filtfilt(b, a, normal.ECG3);

cuf_ECG1_filt = filtfilt(b, a, cuff.ECG1);
cuf_ECG2_filt = filtfilt(b, a, cuff.ECG2);
cuf_ECG3_filt = filtfilt(b, a, cuff.ECG3);

%% Full ECG Signal Plotting

% Normal
time_str = normal.ElapsedTime;
time_duration = duration(time_str, 'Format', "hh:mm:ss.SSS");
time_seconds = seconds(time_duration);
m
figure(1)
plot(time_seconds, nor_ECG1_filt, 'r')
hold on
plot(time_seconds, nor_ECG2_filt, 'g')
plot(time_seconds, nor_ECG3_filt, 'b')
hold off
xlim([10 30])
grid on

%%
% Cuff

figure(2)
time_str2 = cuff.ElapsedTime;
time_duration2 = duration(time_str2, 'Format', "hh:mm:ss.SSS");
time_seconds2 = seconds(time_duration2);

plot(time_seconds2, cuf_ECG1_filt,'r')
hold on
plot(time_seconds2, cuf_ECG2_filt,'g')
plot(time_seconds2, cuf_ECG3_filt,'b')
xlim([10 30])
grid on

%% ECG2 VS Time
figure(3);
plot(time_seconds, nor_ECG2_filt, 'g');
xlim([20 25]); 
grid on;

pbaspect([2 1 1]);  % This makes the x-axis twice as wide as the y-axis


%%
% Extract PPG Signal
nor_PPG = normal.PPGPOX;
cuf_PPG = cuff.PPGPOX;

% Extract Pulse Oximeter Heart Rate
nor_HR_POX = normal.HeartRatePOX;
cuf_HR_POX = cuff.HeartRatePOX;

% Time conversion
time_seconds = seconds(duration(normal.ElapsedTime, 'Format', "hh:mm:ss.SSS"));
time_seconds2 = seconds(duration(cuff.ElapsedTime, 'Format', "hh:mm:ss.SSS"));

%% Define the best lead ECG2
nor_ECG_filt = filtfilt(b, a, normal.ECG2);
cuf_ECG_filt = filtfilt(b, a, cuff.ECG2);

%% Peak Detection and Heart Rate Calculation

% ECG Peaks
[peaks_ECG_nor, locs_ECG_nor] = findpeaks(nor_ECG_filt, time_seconds, 'MinPeakHeight', mean(nor_ECG_filt) + std(nor_ECG_filt), 'MinPeakDistance', 0.6); % 0.6s ~ 100 BPM
[peaks_ECG_cuf, locs_ECG_cuf] = findpeaks(cuf_ECG_filt, time_seconds2, 'MinPeakHeight', mean(cuf_ECG_filt) + std(cuf_ECG_filt), 'MinPeakDistance', 0.6);

% Calculate Heart Rate from ECG
HR_ECG_nor = 60 ./ diff(locs_ECG_nor);
HR_ECG_cuf = 60 ./ diff(locs_ECG_cuf);

% PPG Peaks
[peaks_PPG_nor, locs_PPG_nor] = findpeaks(nor_PPG, time_seconds, 'MinPeakHeight', mean(nor_PPG) + std(nor_PPG), 'MinPeakDistance', 0.6);
[peaks_PPG_cuf, locs_PPG_cuf] = findpeaks(cuf_PPG, time_seconds2, 'MinPeakHeight', mean(cuf_PPG) + std(cuf_PPG), 'MinPeakDistance', 0.6);

% Calculate Heart Rate from PPG
HR_PPG_nor = 60 ./ diff(locs_PPG_nor);
HR_PPG_cuf = 60 ./ diff(locs_PPG_cuf);

%% Plot Data with Peaks Detected

figure(4)
tiledlayout(3,2,"TileSpacing","compact","Padding","compact");

% Normal ECG + Peaks
nexttile;
plot(time_seconds, nor_ECG_filt, 'b');
hold on;
plot(locs_ECG_nor, peaks_ECG_nor, 'ro', 'MarkerSize', 5, 'LineWidth', 1.5);
hold off;
title('Normal ECG with Detected Peaks');
xlabel('Time (s)');
ylabel('ECG Signal');
xlim([10 max(time_seconds)]);  % Show data after 10 seconds

% Cuff ECG + Peaks
nexttile;
plot(time_seconds2, cuf_ECG_filt, 'b');
hold on;
plot(locs_ECG_cuf, peaks_ECG_cuf, 'ro', 'MarkerSize', 5, 'LineWidth', 1.5);
hold off;
title('Cuff ECG with Detected Peaks');
xlabel('Time (s)');
ylabel('ECG Signal');
xlim([10 max(time_seconds2)]);  % Show data after 10 seconds

% Normal PPG + Peaks
nexttile;
plot(time_seconds, nor_PPG, 'g');
hold on;
plot(locs_PPG_nor, peaks_PPG_nor, 'mo', 'MarkerSize', 5, 'LineWidth', 1.5);
hold off;
title('Normal PPG with Detected Peaks');
xlabel('Time (s)');
ylabel('PPG Signal');
xlim([10 max(time_seconds)]);  % Show data after 10 seconds

% Cuff PPG + Peaks
nexttile;
plot(time_seconds2, cuf_PPG, 'g');
hold on;
plot(locs_PPG_cuf, peaks_PPG_cuf, 'mo', 'MarkerSize', 5, 'LineWidth', 1.5);
hold off;
title('Cuff PPG with Detected Peaks');
xlabel('Time (s)');
ylabel('PPG Signal');
xlim([10 max(time_seconds2)]);  % Show data after 10 seconds

% Normal Oxygen Saturation
nexttile;
plot(time_seconds, normal.SpO2POX, 'k');
title('Normal Oxygen Saturation');
xlabel('Time (s)');
ylabel('SpO2 (%)');
xlim([10 max(time_seconds)]);  % Show data after 10 seconds

% Cuff Oxygen Saturation
nexttile;
plot(time_seconds2, cuff.SpO2POX, 'k');
title('Cuff Oxygen Saturation');
xlabel('Time (s)');
ylabel('SpO2 (%)');
xlim([10 max(time_seconds2)]);  % Show data after 10 seconds


%% Compare Heart Rate Estimates
figure(5)
tiledlayout(2,1);

% Normal Condition HR Comparison
nexttile;
plot(locs_ECG_nor(2:end), HR_ECG_nor, 'b', 'DisplayName', 'ECG HR');
hold on;
plot(locs_PPG_nor(2:end), HR_PPG_nor, 'g', 'DisplayName', 'PPG HR');
plot(time_seconds, nor_HR_POX, 'r', 'DisplayName', 'Pulse Ox HR');
hold off;
title('Normal Condition Heart Rate Comparison');
xlabel('Time (s)');
ylabel('Heart Rate (BPM)');
legend;

% Cuff Condition HR Comparison
nexttile;
plot(locs_ECG_cuf(2:end), HR_ECG_cuf, 'b', 'DisplayName', 'ECG HR');
hold on;
plot(locs_PPG_cuf(2:end), HR_PPG_cuf, 'g', 'DisplayName', 'PPG HR');
plot(time_seconds2, cuf_HR_POX, 'r', 'DisplayName', 'Pulse Ox HR');
hold off;
title('Cuff Condition Heart Rate Comparison');
xlabel('Time (s)');
ylabel('Heart Rate (BPM)');
legend;

%% Spirometer 
% Extract the spirometer data columns
time_str = spi.ElapsedTime;  
flow_rate = spi.FlowRateSensor1;  % ml/s unit

% Convert time to seconds
time_duration = duration(time_str, 'Format', 'hh:mm:ss.SSS');
time_seconds = seconds(time_duration);

% Calculate time intervals (differences between consecutive time points)
time_intervals = diff(time_seconds);  

% Calculate volume for each interval by multiplying flow rate and time interval
volume_intervals = flow_rate(1:end-1) .* time_intervals;  % Volume in milliliters

% Calculate the cumulative volume over time
cumulative_volume = cumsum(volume_intervals);  % Volume in milliliters

% Plot the cumulative volume vs. time
figure;
plot(time_seconds(2:end), cumulative_volume / 1000);  % Convert volume to liters
xlabel('Time (seconds)');
ylabel('Volume (liters)');
title('Spirometer Data: Volume vs. Time');
grid on;

% Sum the volumes over all intervals to get the total VC (Vital Capacity)
VC_ml = sum(volume_intervals); 
VC = VC_ml / 1000;  % Convert to liters

% Output the Measured Vital Capacity (VC) in liters
disp(['Measured Vital Capacity: ', num2str(VC), ' L']);
