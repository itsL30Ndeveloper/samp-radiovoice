```php
#include <a_samp>
#include <zcmd>
#include <sampvoice>
#include <sscanf2>

// if you want to make filterscript then you have to define code as filterscript and you must to change OnGameModeInit to OnFilterScriptInit 

#define MAX_RADIOS 9999 //maximum radio frequency channel

new SV_GSTREAM:RadioChannel[MAX_RADIOS] = SV_NULL; 
new FreqNumber[MAX_PLAYERS]; //frequency number
new SV_GSTREAM:gstream = SV_NULL;
new SV_LSTREAM:lstream[MAX_PLAYERS] = { SV_NULL, ... };

/*
    The public OnPlayerActivationKeyPress and OnPlayerActivationKeyRelease
    are needed in order to redirect the player's audio traffic to the
    corresponding streams when the corresponding keys are pressed.
*/

public SV_VOID:OnPlayerActivationKeyPress(SV_UINT:playerid, SV_UINT:keyid) 
{
    // Attach player to local stream as speaker if 'B' key is pressed
    if (keyid == 0x42 && lstream[playerid]) SvAttachSpeakerToStream(lstream[playerid], playerid);
    // Attach the player to the global stream as a speaker if the 'Z' key is pressed
    if (keyid == 0x5A && gstream) SvAttachSpeakerToStream(gstream, playerid);
    // Attach player to radio stream as speaker when 'B' button is pressed
    if(keyid == 0x42 && FreqNumber[playerid] >= 1)
    {
        SvAttachSpeakerToStream(RadioChannel[FreqNumber[playerid]], playerid); // You can add animation and sound variations here to enhance the ingame | Animations : https://sampwiki.blast.hk/wiki/Animations | Sounds : https://sampwiki.blast.hk/wiki/Sound_IDs 
        SetPlayerSpecialAction(playerid,SPECIAL_ACTION_USECELLPHONE); // Animations : Use Radio
        SetPlayerAttachedObject(playerid,0,19942,6); // Radio Object
    }
    if(keyid == 0x42 && FreqNumber[playerid] == 0) return;
}

public SV_VOID:OnPlayerActivationKeyRelease(SV_UINT:playerid, SV_UINT:keyid)
{
    // Detach the player from the local stream if the 'B' key is released
    if (keyid == 0x42 && lstream[playerid]) SvDetachSpeakerFromStream(lstream[playerid], playerid);
    // Detach the player from the global stream if the 'Z' key is released
    if (keyid == 0x5A && gstream) SvDetachSpeakerFromStream(gstream, playerid);
    // Detach the player from the radio stream if the 'B' key is released
    if(keyid == 0x42 && FreqNumber[playerid] >= 1)
    {
        SvDetachSpeakerFromStream(RadioChannel[FreqNumber[playerid]], playerid); // You can add animation and sound variations here to enhance the ingame | Animations : https://sampwiki.blast.hk/wiki/Animations | Sounds : https://sampwiki.blast.hk/wiki/Sound_IDs 
        SetPlayerSpecialAction(playerid,SPECIAL_ACTION_USECELLPHONE); // Animations : Use Radio
        SetPlayerAttachedObject(playerid,0,19942,6); // Radio Object
    }
    if(keyid == 0x42 && FreqNumber[playerid] == 0) return;
}

public OnPlayerConnect(playerid)
{
    // Checking for plugin availability
    if (SvGetVersion(playerid) == SV_NULL)
    {
        SendClientMessage(playerid, -1, "Could not find plugin sampvoice.");
    }
    // Checking for a microphone
    else if (SvHasMicro(playerid) == SV_FALSE)
    {
        SendClientMessage(playerid, -1, "The microphone could not be found.");
    }
    // Create a local stream with an audibility distance of 40.0, an unlimited number of listeners
    // and the name 'Local' (the name 'Local' will be displayed in red in the players' speakerlist)
    else if ((lstream[playerid] = SvCreateDLStreamAtPlayer(40.0, SV_INFINITY, playerid, 0xff0000ff, "Local")))
    {
        SendClientMessage(playerid, -1, "Press Z to talk to global chat and B to talk to local chat.");

        // Attach the player to the global stream as a listener
        if (gstream) SvAttachListenerToStream(gstream, playerid);

        // Assign microphone activation keys to the player
        SvAddKey(playerid, 0x42);
        SvAddKey(playerid, 0x5A);
    }
}

public OnPlayerDisconnect(playerid, reason)
{
    // Removing the player's local stream after disconnecting
    if (lstream[playerid])
    {
        SvDeleteStream(lstream[playerid]);
        lstream[playerid] = SV_NULL;
    }
}

public OnGameModeInit()
{
    // Uncomment the line to enable debug mode
    // SvDebug(SV_TRUE);

    gstream = SvCreateGStream(0xffff0000, "Global");
}

public OnGameModeExit()
{
    if (gstream) SvDeleteStream(gstream);
}

CMD:setfreq(playerid, params[])
{
  new id;
  if(sscanf(params, "d", id)) return SendClientMessage(playerid, -1, "Use: /setfreq [frequensi] | 0 = turn off");

  FreqNumber[playerid] = id;

  if(RadioChannel[FreqNumber[id]] == SV_NULL)
  {
    RadioChannel[FreqNumber[id]] = SvCreateGStream(0xFF00FFFF, "Radio");
    SvAttachListenerToStream(RadioChannel[FreqNumber[id]], playerid);
  }
  else
  {
      SvAttachListenerToStream(RadioChannel[FreqNumber[id]], playerid);
  }
  return 1;
}

```
