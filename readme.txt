export const GetAction = (planSelected, perksSelecetd, selectedPerks, history, storeDetails) => {
  const { SessionValues: { planSelectionJourney } = {} } = storeDetails || {};
  const selectedPerksTerms = Array.isArray(selectedPerks) && selectedPerks?.map((perk) => perk?.spoId);
  const dispatch = useDispatch();
  const lineActivity = `${getIntendType()}`;
  const { staticContent } = storeDetails?.plansReferenceData?.output || {};
  const { contextInfo } = storeDetails?.progressivePlans?.progressivePlanAPiResponse?.data || {};
  const lineLevelInfo = storeDetails?.progressivePlans?.progressivePlanAPiResponse?.data?.lineLevelPlans
    ? storeDetails?.progressivePlans?.progressivePlanAPiResponse?.data?.lineLevelPlans
    : {};
  const plans = (lineLevelInfo && lineLevelInfo?.lines?.length > 0 && lineLevelInfo?.lines[0]?.products[0]?.plans) || [];
  const availablePerks = lineLevelInfo?.lines?.[0]?.availablePerks ?? [];
  const selectedPlanInfo = storeDetails?.progressivePlans?.selectedPlanInfo || {};
  const { getfwaplansLoaded } = storeDetails?.progressivePlans || {};
  const nbxFeedBackCall = () => {
    if (
      getfwaplansLoaded &&
      (selectedPlanInfo ||
        (plans?.length > 0 && selectedPlanInfo?.selectedPlan?.length > 0) ||
        (selectedPerks?.length > 0 && availablePerks?.length > 0))
    ) {
      const selectedPlan = plans?.filter((el) => selectedPlanInfo?.selectedPlan?.planId === el.planId);
      const selectedPerkList = availablePerks?.filter((el) => selectedPerks?.some((f) => f.spoId === el.spoId));
      const selectedPlanPerkList = selectedPlanInfo ? [...selectedPlan, ...selectedPerkList] : selectedPerkList;
      const feedBackTile =
        selectedPlanPerkList &&
        selectedPlanPerkList?.length > 0 &&
        selectedPlanPerkList.map((item) => {
          const isPlan = item?.planId;
          return {
            ...item,
            tacticLocation: isPlan ? 'AvailablePlans' : 'AvailablePerks',
          };
        });
      dispatch({
        type: actionTypes.FEEDBACK_API,
        feedBackTile,
        mtn: 'newLine1',
        dispositionOptionId: 81,
        lineActivityType: lineActivity,
        selectedLLP: '',
        feedbackContextInfo: staticContent?.feedbackContextInfo,
        nbxSessionId: contextInfo?.sessionId,
      });
    }
  };
  const action = {
    helperText: {
      children: perksSelecetd ? (
        <>
          By continuing, you agree to perk{' '}
          <TextLink
            data-testId="terms-and-conditions"
            data-track='{"type": "link","name": "perks tnc"}'
            onClick={() => {
              dispatch({ type: actionTypes.SET_TERMS_AND_CONDITIONS_OVERLAY, response: { selectedPerksTerms, show: true } });
            }}
          >
            Terms & Conditions
          </TextLink>
          .
        </>
      ) : (
        <></>
      ),
    },
    buttonGroup: {
      data: [
        {
          use: 'primary',
          children: 'Continue',
          disabled: false,
          onClick: () => {
            if (!planSelected) {
              eventDispatcher('notify', { name: 'Please select your plan to continue' });
              eventDispatcher('openView', { name: 'message displayed' });
              dispatch({ type: actionTypes.UPDATE_ERROR_NOTIFICATION, payload: true });
            } else {
              if (staticContent?.FeedbackApiFlag) {
                nbxFeedBackCall();
              }
              const journeyCode = planSelectionJourney === PROGRESSIVE ? planJourneyCodes.PROGRESSIVE : planJourneyCodes.POPULAR_CUSTOMIZED;
              dispatch({ type: actionTypes.ADD_PLAN_API, payload: 'progressivePlan', history, planSelectionJourney: journeyCode });
            }
          },
        },
      ],
    },
  };
  return action;
};
