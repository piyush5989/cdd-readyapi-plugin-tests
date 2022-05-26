package com.agimatec.nabaztag.teamcity;

import com.agimatec.nabaztag.Nabaztag;
import jetbrains.buildServer.Build;
import jetbrains.buildServer.BuildType;
import jetbrains.buildServer.notification.Notificator;
import jetbrains.buildServer.notification.NotificatorRegistry;
import jetbrains.buildServer.serverSide.SBuild;
import jetbrains.buildServer.serverSide.SBuildType;
import jetbrains.buildServer.serverSide.SRunningBuild;
import jetbrains.buildServer.serverSide.UserPropertyInfo;
import jetbrains.buildServer.users.NotificatorPropertyKey;
import jetbrains.buildServer.users.PropertyKey;
import jetbrains.buildServer.users.SUser;
import jetbrains.buildServer.vcs.SelectPrevBuildPolicy;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Random;
import java.util.Set;

/**
 * This teamcity plugins is configured as a new notificator. The notifications are text
 * messages, which are sended to the text to speech engine of the Nabaztag.
 * For each teamcity user it is possible to configure his Nabaztag settings and notification events. In this
 * way multiple Nabaztags could be accessed for different projects or developers.
 * The configuration could be found in the same place, where mail or ide notifications are configured.
 * <p/>
 * User: Simon Tiffert
 * Copyright: Agimatec GmbH 2009
 * <p/>
 * Changes made by Daniel Wellman (dwellman@cyrusinnovation.com):
 * - Renamed the Rabbit ID display field to clearly indicate it's the serial number
 * - Fixed a spelling error in successful
 * - You can now specify a voice to use (e.g. UK-Penelope).  For a list of
 *      voices, send your rabbit this command:
 *      http://api.nabaztag.com/vl/FR/api.jsp?sn=MYSERIALNUMBER&token=MYTOKEN&action=9
 *
 * Changes made by Robert Moran:
 * - Exposed all build messages to the user for customisation
 * - Added #PROJECT#, #USER# and #COMMENT# placeholders which can be used
 *      in the messages and are replaced with the project name, user or username of
 *      the last person to make a change and their commit comments
 *
 * - if no voice is specified a random voice is picked
 */
public class NabaztagNotificator implements Notificator {

    private static final String TYPE = "nabaztagNotifier";
    private static final String TYPE_NAME = "Nabaztag Notifier";
    private static final String NABAZTAG_RABBIT_ID = "rabbitId";
    private static final String NABAZTAG_RABBIT_TOKEN = "rabbitToken";
    private static final String NABAZTAG_RABBIT_VOICE = "rabbitVoice";
    private static final String NABAZTAG_BUILD_STARTED = "buildStarted";
    private static final String NABAZTAG_BUILD_SUCCESSFUL = "buildSuccessful";
    private static final String NABAZTAG_BUILD_FAILED = "buildFailed";
    private static final String NABAZTAG_BUILD_LABELING = "buildLabeling";
    private static final String NABAZTAG_BUILD_FAILING = "buildFailing";
    private static final String NABAZTAG_BUILD_HANGING = "buildHanging";
    private static final String NABAZTAG_BUILD_RESPONSIBLE = "buildResponsible";

    private static final PropertyKey RABBIT_ID = new NotificatorPropertyKey(TYPE, NABAZTAG_RABBIT_ID);
    private static final PropertyKey RABBIT_TOKEN = new NotificatorPropertyKey(TYPE, NABAZTAG_RABBIT_TOKEN);
    private static final PropertyKey RABBIT_VOICE = new NotificatorPropertyKey(TYPE, NABAZTAG_RABBIT_VOICE);
    private static final PropertyKey BUILD_STARTED = new NotificatorPropertyKey(TYPE, NABAZTAG_BUILD_STARTED);
    private static final PropertyKey BUILD_SUCCESSFUL = new NotificatorPropertyKey(TYPE, NABAZTAG_BUILD_SUCCESSFUL);
    private static final PropertyKey BUILD_FAILED = new NotificatorPropertyKey(TYPE, NABAZTAG_BUILD_FAILED);
    private static final PropertyKey BUILD_LABELING = new NotificatorPropertyKey(TYPE, NABAZTAG_BUILD_LABELING);
    private static final PropertyKey BUILD_FAILING = new NotificatorPropertyKey(TYPE, NABAZTAG_BUILD_FAILING);
    private static final PropertyKey BUILD_HANGING = new NotificatorPropertyKey(TYPE, NABAZTAG_BUILD_HANGING);
    private static final PropertyKey BUILD_RESPONSIBLE = new NotificatorPropertyKey(TYPE, NABAZTAG_BUILD_RESPONSIBLE);

    private static final String DEFAULT_STARTED_MESSAGE = "Build #PROJECT# started.";
    private static final String DEFAULT_SUCCESSFUL_MESSAGE = "Build #PROJECT# successfull.";
    private static final String DEFAULT_FAILED_MESSAGE = "Build #PROJECT# failed.";
    private static final String DEFAULT_LABELING_MESSAGE = "Labeling of build #PROJECT# failed.";
    private static final String DEFAULT_FAILING_MESSAGE = "Build #PROJECT# is failing.";
    private static final String DEFAULT_HANGING_MESSAGE = "Build #PROJECT# is probably hanging.";
    private static final String DEFAULT_RESPONSIBLE_MESSAGE = "Responsibility of build #PROJECT# changed.";

    public NabaztagNotificator(NotificatorRegistry notificatorRegistry) throws IOException {
        ArrayList<UserPropertyInfo> userProps = new ArrayList<UserPropertyInfo>();
        userProps.add(new UserPropertyInfo(NABAZTAG_RABBIT_ID, "Nabaztag Serial #"));
        userProps.add(new UserPropertyInfo(NABAZTAG_RABBIT_TOKEN, "Nabaztag Token"));
        userProps.add(new UserPropertyInfo(NABAZTAG_RABBIT_VOICE, "Nabaztag Voice (optional)"));
        userProps.add(new UserPropertyInfo(NABAZTAG_BUILD_STARTED, "Started Message"));
        userProps.add(new UserPropertyInfo(NABAZTAG_BUILD_SUCCESSFUL, "Success Message"));
        userProps.add(new UserPropertyInfo(NABAZTAG_BUILD_FAILED, "Failed Message"));
        userProps.add(new UserPropertyInfo(NABAZTAG_BUILD_LABELING, "Labeling Message"));
        userProps.add(new UserPropertyInfo(NABAZTAG_BUILD_FAILING, "Failing Message"));
        userProps.add(new UserPropertyInfo(NABAZTAG_BUILD_HANGING, "Hanging Message"));
        userProps.add(new UserPropertyInfo(NABAZTAG_BUILD_RESPONSIBLE, "Changed Message"));
        notificatorRegistry.register(this, userProps);
    }

    public void notifyBuildStarted(SRunningBuild sRunningBuild, Set<SUser> sUsers) {
        doNotifications(sRunningBuild, sUsers, null, BUILD_STARTED, DEFAULT_STARTED_MESSAGE);
    }

    public void notifyBuildSuccessful(SRunningBuild sRunningBuild, Set<SUser> sUsers) {
        doNotifications(sRunningBuild, sUsers, Nabaztag.EARS_HAPPY, BUILD_SUCCESSFUL, DEFAULT_SUCCESSFUL_MESSAGE);
    }

    public void notifyBuildFailed(SRunningBuild sRunningBuild, Set<SUser> sUsers) {
        doNotifications(sRunningBuild, sUsers, Nabaztag.EARS_SAD, BUILD_FAILED, DEFAULT_FAILED_MESSAGE);
    }

    public void notifyLabelingFailed(Build build, jetbrains.buildServer.vcs.VcsRoot vcsRoot, Throwable throwable, Set<SUser> sUsers) {
        doNotifications(build, sUsers, Nabaztag.EARS_SAD, BUILD_LABELING, DEFAULT_LABELING_MESSAGE);
    }

    public void notifyBuildFailing(SRunningBuild sRunningBuild, Set<SUser> sUsers) {
        doNotifications(sRunningBuild, sUsers, Nabaztag.EARS_SAD, BUILD_FAILING, DEFAULT_FAILING_MESSAGE);
    }

    public void notifyBuildProbablyHanging(SRunningBuild sRunningBuild, Set<SUser> sUsers) {
        doNotifications(sRunningBuild, sUsers, Nabaztag.EARS_SAD, BUILD_HANGING, DEFAULT_HANGING_MESSAGE);
    }

    public void notifyResponsibleChanged(SBuildType sBuildType, Set<SUser> sUsers) {
        doNotifications(sBuildType, sUsers, null, BUILD_RESPONSIBLE, DEFAULT_RESPONSIBLE_MESSAGE);
    }

    public String getNotificatorType() {
        return TYPE;
    }

    public String getDisplayName() {
        return TYPE_NAME;
    }

    private void doNotifications(Build build, Set<SUser> sUsers, String rabbitEars, PropertyKey key, String defaultValue) {
        for (SUser user : sUsers) {
            String message = user.getPropertyValue(key);
            if (message == null || message.equals("")) {
                message = defaultValue;
            }
            message = replaceText(message, "#PROJECT#", build.getFullName());
            message = replaceText(message, "#USER#", getUser(build));
            message = replaceText(message, "#COMMENT#", getComment(build));
            doNotification(message, user, rabbitEars);
        }
    }

    private void doNotifications(BuildType buildType, Set<SUser> sUsers, String rabbitEars, PropertyKey key, String defaultValue) {
        for (SUser user : sUsers) {
            String message = user.getPropertyValue(key);
            if (message == null || message.equals("")) {
                message = defaultValue;
            }
            message = replaceText(message, "#PROJECT#", buildType.getFullName());
            message = replaceText(message, "#USER#", buildType.getResponsibilityInfo().getUser().getName());
            message = replaceText(message, "#COMMENT#", buildType.getResponsibilityInfo().getComment());
            doNotification(message, user, rabbitEars);
        }
    }

    public void doNotification(String message, SUser user, String rabbitEars) {
        Nabaztag nabaztag = new Nabaztag();

        nabaztag.setRabbitID(user.getPropertyValue(RABBIT_ID));
        nabaztag.setToken(user.getPropertyValue(RABBIT_TOKEN));
        nabaztag.setText(message);

        String voice = user.getPropertyValue(RABBIT_VOICE);

        if (voice != null && !voice.equals("")) {
            nabaztag.setVoice(voice);
        }
        else {
            nabaztag.setVoice(getRandomVoice());
        }
        if (rabbitEars != null) {
            nabaztag.setEars(rabbitEars);
        }

        nabaztag.publish();

    }

    private String replaceText(String source, String find, String replace) {
        String result = source;

        if (replace != null && !replace.equals("")) {
            result = result.replaceAll(find, replace);
        }

        return result;
    }

    private String getUser(Build build) {
        String user = "";

        try {
            if (build instanceof SBuild) {
                SBuild sBuild = (SBuild) build;
                // TODO new Teamcity API?
                /*if (sBuild.getTriggeredBy().isTriggeredByUser()) {
                    user = sBuild.getTriggeredBy().getUser().getName();
                }*/
            }
        }
        catch (Exception e) {
        }

        if (user == null || user.equals("")) {
            try {
                user = build.getCommitters(SelectPrevBuildPolicy.SINCE_LAST_BUILD).getUsers().iterator().next().getName();
            }
            catch (Exception e) {
            }
        }

        if (user == null || user.equals("")) {
            try {
                user = build.getContainingChanges().get(0).getUserName();
            }
            catch (Exception e) {
            }
        }

        return user;
    }

    private String getComment(Build build) {
        String comment = "";

        try {
            comment = build.getContainingChanges().get(0).getDescription();
        }
        catch (Exception e) {
        }

        return comment;
    }

    private String getRandomVoice() {
        String[] voices = new String[]{"AU-Colleen", "AU-Jon", "UK-Edwin", "UK-Leonard", "UK-Mistermuggles", "UK-Penelope", "UK-Rachel", "UK-Shirley", "US-Bethany", "US-Billye", "US-Clarence", "US-Darleen", "US-Ernest", "US-Liberty", "US-Lilian"};
        int index = new Random().nextInt(voices.length);
        return voices[index];
    }
}
