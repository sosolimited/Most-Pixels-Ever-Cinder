![MPE](https://raw.github.com/wdlindmeier/Most-Pixels-Ever-Cinder/master/assets/MPEIcon.gif) #Most-Pixels-Ever for Cinder

A Cinder block client for Most Pixels Ever.

Most Pixels Ever is a framework that synchronizes frame-based applications across 
multiple screens.

Read more about the project:
https://github.com/shiffman/Most-Pixels-Ever

###Example Usage:

#####The Server

A basic MPE 2.0 server is included with the block. Run like so:

`$ python server/simple_server.py`

#####Your Cinder App

```c++
class MyCinderApp : public AppNative, public MPEApp
{
    public:      
    
    void        setup();    
    void        update();
    void        draw();
    
    // Override functions found in MPEApp
    void        mpeReset();
    void        mpeFrameUpdate(long serverFrameNumber);
    void        mpeFrameRender(bool isNewFrame);
    void        mpeMessageReceived(const std::string & message, const int fromClientID);
    
    private:
    
    MPEAsyncClient *mClient;      
    Rand            mRand;
    BouncyBall      mBall;
}

void MyCinderApp::setup()
{
    // By default, the client is be configured using assets/settings.xml.
    // The settings filename is changed by overriding MPEApp::mpeSettingsFilename(). 
    // Each client needs a unique settings file.
    // Pass a pointer to an MPEApp (e.g. MyCinderApp) into the client constructor.
    
    mClient = new MPEAsyncClient(this);
}

void MyCinderApp::update()
{
    // Connect the client if we're not already.
    // Nothing else needs to happen in update().
    // Updating the App state happens in mpeFrameUpdate().
    
    if (!mClient->isConnected() && getElapsedFrames() % 60 == 0)
    {
        // Attempt to reconnect every 60 frames.
        mClient->start();
    }
}

void MyCinderApp::draw()
{
    // App drawing should happen in mpeFrameRender(). 
    
    mClient->draw();
}

// functions overridden from MPEApp

void MyCinderApp::mpeReset()
{
    // This is called whenever a new client joins the loop.
    // All state and history should be cleared as if the app 
    // was just launched. This will be called by the client
    // after it connects to the server.
    
    mRand.seed(1);
    // This will create the same "random" position each reset because the 
    // seed is hardcoded to 1.
    Vec2i sizeMaster = mClient->getMasterSize();
    mBall.position = Vec2f(mRand.nextFloat(sizeMaster.x), mRand.nextFloat(sizeMaster.y))
    mBall.velocity = Vec2f(mRand.nextFloat(-5,5), mRand.nextFloat(-5,5));
}

void MyCinderApp::mpeFrameUpdate(long serverFrameNumber)
{
    // This is where the app state should be modified. 
    // The FrameUpdateCallback is called whenever we get a message from the server,
    // which may be less frequently than update() or draw() is called.    
    
    mBall.update();
}

void MyCinderApp::mpeFrameRender(bool isNewFrame)
{
    // The viewport is automatically translated by the client so each
    // machine is only drawing a segment of the scene.
    
    gl::clear(Color(0,0,0));
    mBall.draw();
}

void MyCinderApp::mpeMessageReceived(const std::string & message, const int fromClientID)
{
    // Apps can broadcast data to the other clients. E.g.:
    //
    // mClient->sendMessage("mouse_pos: 100,200");

    ci::app::console() << "Client " << fromClientID 
                       << " sent broadcast message: " << message << std::endl;
}
```