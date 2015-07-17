#include <string>

#include <yarp/os/Network.h>
#include <yarp/os/RFModule.h>
#include <yarp/os/RateThread.h>
#include <yarp/os/BufferedPort.h>
#include <yarp/os/Time.h>
#include <yarp/sig/Vector.h>
#include <cstdlib>   
#include <yarp/dev/Drivers.h>
#include <yarp/dev/GazeControl.h>
#include <yarp/dev/PolyDriver.h>

YARP_DECLARE_DEVICES(icubmod)

using namespace std;
using namespace yarp::os;
using namespace yarp::dev;
using namespace yarp::sig;


class CtrlThread: public RateThread
{
protected:
    ResourceFinder &rf;

    PolyDriver      clientGaze;
    IGazeControl   *igaze;

    int state;
    int startup_context_id;


public:
    CtrlThread(ResourceFinder &_rf) : RateThread(20), rf(_rf) { }

    bool threadInit()
    {
        string name=rf.find("name").asString().c_str();
        double period=rf.find("period").asDouble();
        setRate(int(1000.0*period));

        Property optGaze("(device gazecontrollerclient)");
        optGaze.put("remote","/iKinGazeCtrl");
        optGaze.put("local",("/"+name+"/gaze").c_str());

        if (!clientGaze.open(optGaze))
            return false;

        clientGaze.view(igaze);

        igaze->storeContext(&startup_context_id);

        igaze->setNeckTrajTime(0.8);
        igaze->setEyesTrajTime(0.4);

        return true;
    }
    int generate_rand_pos(int LOW, int HIGH)

    {

    return rand() % (HIGH - LOW + 1) + LOW;

    }
    void run()
    {
  double randno = (double) generate_rand_pos(90,100)/100; 
  igaze->setNeckTrajTime(randno);
  igaze->setEyesTrajTime(randno);
    Vector ang(3);
    ang[0]= generate_rand_pos(-15,15); 
    ang[1]= generate_rand_pos(-10,10);
    ang[2]= 5;
    igaze->lookAtAbsAngles(ang); 
   igaze->waitMotionDone(2);
    }

    void threadRelease()
    {    
        igaze->stopControl();
        igaze->restoreContext(startup_context_id);

        clientGaze.close();
    }
};


class CtrlModule: public RFModule
{
protected:
    CtrlThread *thr;

public:
    bool configure(ResourceFinder &rf)
    {
        Time::turboBoost();
        thr=new CtrlThread(rf);
        if (!thr->start())
        {
            delete thr;
            return false;
        }

        return true;
    }

    bool close()
    {
        thr->stop();
        delete thr;

        return true;
    }

    double getPeriod()    { return 1.0;  }
    bool   updateModule() { return true; }
};



int main(int argc, char *argv[])
{
    // we need to initialize the drivers list 
    YARP_REGISTER_DEVICES(icubmod)

    Network yarp;
    if (!yarp.checkNetwork())
        return -1;    

    ResourceFinder rf;
    rf.setVerbose(true);
    rf.setDefault("name","mvrand");
    rf.setDefault("period","0.02");
    rf.configure(argc,argv);

    CtrlModule mod;
    return mod.runModule(rf);
}



