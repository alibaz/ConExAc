# ConExAc
Concurrent Excitation and Acquisition Reconstruction


Pseudo-code for the CEA data processing and gridding reconstruction
Read_data(‘directory name’)
%All the CEA raw data files in the given directory are read and the required variables are stored in .mat files:
%[rawread, loopcounters, sMDH,radialangles]=ReadMeas(fname);

gridRecon3D(‘Data file name’);

Variables:
%dummyscans
%avg: number of averages
%rmean1=rawread((1+dummyscans):(rawsize(1)-1),:);
%radialangles=0.01*radialangles(:,(1+dummyscans):(dummyscans+(rawsize(1)-dummyscans)/avg));

Data is sorted with the respect to the number of acquired averages and the mean raw data is calculated:
rmean=rmean/avg;
rawsize=size(rmean);

Acquisition parameters are read from the raw data file:
BW: acquisition bandwidth per pixel
gamma=42.576e6 Hz/T
N=rawsize(2);
tADC=linspace(0,1/BW,N) (sec)
GradMaxAmp (mT/m)
maxSlew=320e3 (mT/m/s)
n_TotalProjections=rawsize(1);
gridscalefactor=1;%gridding enhancement constant=>a finer grid. Default:1

display('Readout FOV in cm = ');
FOV=100*N*BW/(gamma*GradMaxAmp*1e-3)
display ('... in  cm');

bw (Hz): FM RF pulse sweep range
Tp (sec): FM RF pulse duration



Post processing on Raw data
[rmean,shift,rawsize]=postproc(       0    ,                   0                 ,                            0                              ,             0           ,  1   ,rmean,radialangles,ch_rx,BW,bw,Tp);
FILTER,GRADIENTDELAY,OPPOSITESPOKEMISMATCH,PHASESHIFT,CEA

Gradient shape calculation
 kr=gradshapecalc(maxSlew,GradMaxAmp,BW,N);

Calculation for the 3D location information of samples on the radial spokes
display('Calculating k space locations from radial spoke angle index ...')
[kx,ky,kz, theta, phi]=ktrajectory_radialangles(GradMaxAmp,n_TotalProjections,kr,radialangles,shift);


Assign the grid location values by scaling the actual kspace locations to integer values by rounding
        tic
        display('Mapping of the data on to the specified Cartesian grid ...')
        [rgridded, rdens]=grid3D_KB(kx,ky,kz,rmean,gridscalefactor,0.5);
        display('Regridding: DONE!')
        toc
 
 
    display('Calculating 3D FFT of the regridded data ...')
    RG=(fftshift(fftn(fftshift(rgridded))));
    RG(isnan(RG))=0;
    display('Calculating FFT: DONE!')
    toc

Display the results

%%%%%%%%%%%%%%%%%%%%%%%%%%%THE END%%%%%%%%%%%%%%%%%%%%%%%%%
