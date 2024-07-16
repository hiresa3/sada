import * as actionTypes from '../@Redux/actionTypes';
import { isJointTransactionFlow, isLoggedIn } from '../../../components/common/Helpers';

export const handleDuplicatePerks = (state, toggleOn, perkInfo, viewedDuplicatePerks, dispatch, isPerkToggleClicked) => {
  let showDisclosurePerk = true;
  const shareablePerks = state?.progressivePlans?.progressivePlanAPiResponse?.data?.shareablePerks
    ? state?.progressivePlans?.progressivePlanAPiResponse?.data?.shareablePerks
    : [];
    
    const shareableperksSpoids = shareablePerks.map((i)=> i.spoId);
    
  if (isJointTransactionFlow() || isLoggedIn()) {
    if (toggleOn) {
      shareableperksSpoids.forEach((val) => {
        if (val === perkInfo.spoId) {
          showDisclosurePerk = false;
          const perkDuplicateOverlay = state?.progressivePlans?.perkDuplicateOverlay ? state?.progressivePlans?.perkDuplicateOverlay : {};
          const disclosureDisplayed = new Set(perkDuplicateOverlay?.disclosureDisplayed);
          if (!disclosureDisplayed.has(perkInfo.spoId)) {
            disclosureDisplayed.add(perkInfo.spoId);
            if (isPerkToggleClicked) {
              if (!viewedDuplicatePerks.has(perkInfo.spoId)) {
                perkDuplicateOverlay.show = true;
                viewedDuplicatePerks.add(perkInfo.spoId);
              }
            } else {
              perkDuplicateOverlay.show = true;
              viewedDuplicatePerks.add(perkInfo.spoId);
            }
            perkDuplicateOverlay.currentSpoId = perkInfo.spoId;
            perkDuplicateOverlay.disclosureDisplayed = Array.from(disclosureDisplayed);
          }
          dispatch({ type: actionTypes.DUPLICATE_PERK_DISCLOSURE, response: perkDuplicateOverlay });
        }
      });
    }
  }
  return showDisclosurePerk;
};

export const handleDisclosurePerks = (perkStaticContent, perkInfo, state, toggleOn, dispatch) => {
  const disclosurePerksSpoIds = [perkStaticContent?.disneyPerkSpoId];
  if (!isLoggedIn()) {
    disclosurePerksSpoIds.push(perkStaticContent?.appleOneIndividualPerkSpoId);
    disclosurePerksSpoIds.push(perkStaticContent?.appleOneFamilyPerkSpoId);
  }
  if (disclosurePerksSpoIds?.includes(perkInfo?.spoId) && toggleOn) {
    const perkDisclosureOverlay = state?.progressivePlans?.perkDisclosureOverlay ? state?.progressivePlans?.perkDisclosureOverlay : {};
    const disclosureDisplayed = new Set(perkDisclosureOverlay?.disclosureDisplayed); // Taking out any duplicate from array available.
    if (!disclosureDisplayed.has(perkInfo?.spoId) && perkInfo?.spoId) {
      disclosureDisplayed.add(perkInfo?.spoId);
      perkDisclosureOverlay.show = true;
      perkDisclosureOverlay.currentSpoId = perkInfo.spoId;
      perkDisclosureOverlay.disclosureDisplayed = Array.from(disclosureDisplayed);
    }
    dispatch({ type: actionTypes.UPDATE_PERK_DISCLOSURE, response: perkDisclosureOverlay });
  }
};

