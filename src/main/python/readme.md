extracts data using following format

    DWORD magic;   // "mvr+"
    DWORD version;  // 4 (or higher)
    DWORD headerSize;  // 7 (or more, in DWORDs)
    DWORD sceneCount;
    DWORD videoDuration;  // in frames
    DWORD flags;  // 0x00000001 = complete
    DWORD maxCLL;  // measured MaxCLL

    Scene Data
    Starts at byte "headerSize * sizeof(DWORD)".
    has 3 segments:
        1) The first frame of the scenes
        2) The last frame of the scenes
        3) The peak nits of the scenes.

    So if e.g. sceneCount says 5, then there are 5 scenes.
    The file will have
    - 5 DWORDs giving you the frame number of the first frame in each scene.
    - 5 DWORDs giving you the frame number (+1) of the last frame in each scene.
    - 5 DWORDs giving you the measured peak of each frame, in Nits.

    Finally, at file offset "headerSize * sizeof(DWORD) + 3 * sceneCount * sizeof(DWORD)", there will be detailed measurements for each video frame.

    The "videoDuration" will define how many of those detailed measurements are in the file.

    For each frame there are 32 WORDs stored.

        1) The first WORD is the measured peak of the frame, in PQ light.
        You need to divide that number by 64000.0, and then decode the PQ transfer function, and then multiply with 10000.0 to get the Nits number.
        2) The other remaining 31 WORDs have the percentage values of each of the 31 histogram bars.
        Divide by 640.0 to get a percentage number from 0.0 - 100.0.


    static double pq2linear(double pq)
    {
      double temp = pow(pq, 1.0 / 78.84375);
      return pow(max(temp - 0.8359375, 0) / (18.8515625 - 18.6875 * temp), 1.0 / 0.1593017578125);
    }

    This function expects the "pq" value to be in the 0.0-1.0 range, and the linear return value will also be in the same range.
    You have to multiply the linear return value with 10000.0 to get the nits number.

