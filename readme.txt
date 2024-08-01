/* eslint-disable react/jsx-curly-newline */
/* eslint-disable radix */

import React from 'react';
import PropTypes from 'prop-types';

import { TitleLockup } from '@vds/type-lockups';

import { isMobile, isTablet, getFlowType, getSession, viewport, checkIsMidnight } from '../../../common/Helpers/index';

import './addons.css';

import { store } from '../../../../configureStore';
import { eventDispatcher, updateEventInfo } from '../../../common/Tagging/index';
import { logMetrics, ACCESSORIES, USER_INFO, ADDTOCART_BUTTON_CLICK, REMOVE_BUTTON_CLICK } from '../../../../services/metricsService';

import { getDepletionType } from '../../../common/CommonUtils';
import StreamingServices from './Components/FunctionalRedesign/streamingService';
import { getAddonsRefData, prepareTilesDataClone as prepareAddonsDataRef } from './Components/FunctionalRedesign/addonsCommon';
import { Margin } from '../../../common/styles';

import { getLqAddress } from '../../../common/addressUtils.js';

export default class StreamingService extends React.PureComponent {
  constructor(props) {
    super(props);
    this.isMobile = isMobile();
    this.isTablet = isTablet();
    this.flowType = getFlowType();
    this.isMidnight = checkIsMidnight();
    this.state = {
      skipStatus: false,
      // notificationCalled: false,
      // hideNotificationCalled: false,
      buttonSelectState: '',
      // buttonTextChange: 'addToCart',
      // disableBtn: { homeDeviceProtect: false, homeDeviceAdvisor: false },
    };
    this.prepareTilesDataClone = prepareAddonsDataRef.bind(this);
  }

  // componentDidMount() {
  //   if (this.props?.cartDetails?.cartDetailsJson !== '') {
  //     this.props.invokeTaggingAPI();
  //   }
  // }

  componentDidUpdate(prevProps, prevState) {
    if (prevState.skipStatus !== this.state.skipStatus) {
      this.props.getskipStatus(this.state.skipStatus);
    }
  }

  changeSkip = (val) => {
    this.setState({ skipStatus: val });
  };

  showNotification = (displayName, count, prodID, prodType) => {
    let verbiage = '';
    // this.setState({ notificationCalled: true });
    // this.setState({ hideNotificationCalled: false });
    if (!this.isMobile) {
      verbiage =
        displayName !== undefined
          ? prodType === 'streamingService'
            ? `${displayName} is added to your cart.`
            : `${count} ${displayName} is added to your cart.`
          : ' ';
      this.props.showAddonNotification(verbiage);
      window.scrollTo(0, 0);
    } else if (this.isMobile) {
      this.props.showAddonNotification('mdot');
    }
  };

  hideNotification = () => {
    this.props.showAddonNotification('');
    this.props.setSelectedProduct('');
    // this.setState({ notificationCalled: false });
  };

  getTileTitle = (planInfo) => planInfo && (planInfo.displayName || planInfo.planName);

  getTileImage = (planInfo) => {
    let image;
    const viewPort = viewport();
    if (planInfo && planInfo.vdsImageUrl) {
      image = (
        <div className={isMobile() ? 'pad60 onlyBottomPad' : 'width100px height66'}>
          <img
            src={planInfo && planInfo.vdsImageUrl}
            alt=" "
            className={planInfo.displayName === 'Stream TV' ? 'vdsUpImageST' : 'vdsUpImageYT'}
            tabIndex={isMobile() || (viewPort === 'mfapp' && '-1')}
          />
        </div>
      );
    }
    return image;
  };

  showPromoText = (planInfo, promoEligible) => {
    let promoText;
    if (promoEligible && planInfo && planInfo[0]) {
      promoText = (
        <p className="fontSize_3 color_blue promoCta promo_cta paddTop2">{promoEligible && planInfo && planInfo[0] && planInfo[0]?.promoText}</p>
      );
    }
    return promoText;
  };

  onRemovingStreamTV = (buttonValue, quantity, dispName) => {
    this.setState({ buttonSelectState: buttonValue });
    if (quantity === 0) {
      this.props.showAddonNotification(`${dispName} is removed from cart.`);
    }
    setTimeout(() => {
      console.log(this.props, 'cart');
      const addToCart = document.getElementById('_AddtoCart');
      if (addToCart && !quantity && this.state.buttonSelectState === 'remove') {
        addToCart.focus();
      }
      this.props.showAddonNotification('');
    }, 6000);
  };

  addToCart = (planInfo, notifyMessage, skuDetails, type, displayName, recurringInd, pricePlanId) => {
    // this.setState({ buttonTextChange: 'removeFromCart' });
    store.dispatch({ type: 'LOADING_DONE', response: false });
    const oddOnList = this.props && this.props && this.props.addonObj ? this.props.addonObj : [];
    if (type === 'streamingDevice') {
      const prodId = skuDetails && skuDetails.sorId ? skuDetails.sorId : skuDetails.productId;
      const qty = '1';
      const depletionType = getDepletionType(this.props.cartDetails?.dueTodayList);
      // const name = this.props.skuDetails && this.props.skuDetails.displayName;
      const data = {
        addAccessories: {
          prodId,
          qty,
        },
        depletionType,
      };
      if (notifyMessage) {
        data.notifyMessage = notifyMessage;
      }
      this.props.postAddToCart(data, type, oddOnList, 'addToCart');
      eventDispatcher('addToCart', data);
      updateEventInfo('scAdd');
      this.setState({ buttonSelectState: 'add' });
      // this.showNotification(name, 0, prodId);
      this.props.setSelectedProduct(prodId);
    } else if (type === 'streamingService') {
      const prodId = skuDetails && skuDetails.sorId ? skuDetails.sorId : skuDetails.productId;
      const price = skuDetails && skuDetails.priceText;
      const name = skuDetails && skuDetails.displayName;
      const data = {
        addYtTv: {
          prodId,
          pricePlanId,
          price,
          name,
          recurringInd,
        },
      };
      if (notifyMessage) {
        data.notifyMessage = notifyMessage;
      }
      this.props.postAddToCart(data, type, oddOnList, 'addToCart');
      eventDispatcher('addToCart', data);
      updateEventInfo('scAdd');
      // this.showNotification(name, 0, prodId);
      this.props.setSelectedProduct(prodId);
      if (this.props) {
        this.props.getskipStatus(true);
      } else if (this.props?.getskipStatus) {
        this.props.getskipStatus(true);
      }
    } else if (type === 'vzProtectHome') {
      const prodId = skuDetails && skuDetails.sorId ? skuDetails.sorId : skuDetails.productId;
      const qty = '1';

      const data = {
        addVZProtect: {
          prodId,
          qty,
        },
      };
      if (notifyMessage) {
        data.notifyMessage = notifyMessage;
      }
      this.props.postAddToCart(data, type, oddOnList, 'addToCart');
      eventDispatcher('addToCart', data);
      updateEventInfo('scAdd');
      // this.showNotification(planName, 0, prodId);
      this.props.setSelectedProduct(prodId);
      if (getSession('isCBandFlow') === 'Y') {
        this.handleBtnClick(displayName, 'addToCart');
      }
    } else if (type === 'homePhoneService' && this.cBand) {
      let zipCode;
      let city;
      let fipsCode;
      let addressLine1;
      let state;

      const lqShippingAddress = getLqAddress();
      if (lqShippingAddress && lqShippingAddress !== undefined) {
        zipCode = lqShippingAddress.zipCode ? lqShippingAddress.zipCode : '';
        city = lqShippingAddress.city ? lqShippingAddress.city : '';
        fipsCode = lqShippingAddress.fipsCode ? lqShippingAddress.fipsCode : '';
        addressLine1 = lqShippingAddress.addressLine1 ? lqShippingAddress.addressLine1 : '';
        state = lqShippingAddress.state ? lqShippingAddress.state : '';
      }
      const depletionType = getDepletionType(this.props.cartDetails?.dueTodayList);
      const data = {
        prodId: this.props.homePhoneID,
        qty: '1',
        planId: this.props.homeProdId,
        zipCode,
        city,
        fipsCode,
        addressLine1,
        state,
        depletionType,
      };
      this.props.postAddToCart(data, type, oddOnList, 'addToCart');
    }
    logMetrics(ACCESSORIES, USER_INFO, ADDTOCART_BUTTON_CLICK);
  };

  removeCart = (planInfo, skuDetails, displayName, type, recurringInd, pricePlanId) => {
    // this.setState({ buttonTextChange: 'addToCart' });
    const oddOnList = this.props && this.props && this.props.addonObj ? this.props.addonObj : [];
    this.props.showAddonNotification(`${displayName} is removed from cart.`);
    if (type === 'streamingService') {
      const prodId = skuDetails && skuDetails.sorId ? skuDetails.sorId : skuDetails.productId;
      const price = skuDetails && skuDetails.priceText;
      const name = skuDetails && skuDetails.displayName;
      const depletionType = getDepletionType(this.props.cartDetails?.dueTodayList);
      const data = {
        removeYtTv: {
          prodId,
          pricePlanId,
          price,
          name,
          recurringInd,
        },
        depletionType,
      };
      this.props.postAddToCart(data, type, oddOnList, 'removeFromCart');
      eventDispatcher('removeFromCart', data);
      if (this.props) {
        this.props.getskipStatus(false);
      } else if (this.props?.getskipStatus) {
        this.props.getskipStatus(false);
      }
      this.setState({ buttonSelectState: 'remove' });
    } else if (type === 'vzProtectHome') {
      const prodId = skuDetails && skuDetails.sorId ? skuDetails.sorId : skuDetails.productId;
      const qty = '1';
      const numOfLines = this.props.cartDetails && this.props.cartDetails.orderDetailsView && this.props.cartDetails.orderDetailsView.numOfLines;
      const data = {
        removeVZProtect: {
          prodId,
          qty,
          numOfLines,
        },
      };
      this.props.postAddToCart(data, type, oddOnList, 'removeFromCart');
      eventDispatcher('removeFromCart', data);
      if (getSession('isCBandFlow') === 'Y') {
        this.props.handleBtnClick(displayName, 'removeToCart');
      }
    } else if (type === 'homePhoneService' && !this.cBand) {
      const npanxxVal = this.props.npanxx;
      const depletionType = getDepletionType(this.props.cartDetails?.dueTodayList);
      const data = {
        npanxx: npanxxVal,
        depletionType,
      };
      this.props.postAddToCart(data, 'removeHomePhone', oddOnList, 'removeFromCart');
      eventDispatcher('removeFromCart', data);
      store.dispatch({ type: 'IS_E911_CALL', value: false });
    } else if (type === 'homePhoneService' && this.cBand) {
      const depletionType = getDepletionType(this.props.cartDetails?.dueTodayList);
      const data = {
        depletionType,
      };
      this.props.postAddToCart(data, 'removeHomePhone', oddOnList, 'removeFromCart');
      eventDispatcher('removeFromCart', data);
    }
    logMetrics(ACCESSORIES, USER_INFO, REMOVE_BUTTON_CLICK);
    setTimeout(() => {
      this.props.showAddonNotification('');
    }, 6000);
  };

  setTimeOutCall = () => {
    setTimeout(() => {
      this.hideNotification();
    }, 6000);
    // this.setState({ hideNotificationCalled: true });
  };

  render() {
    const { title, accessoryDetails, subTitle, addOnLoadingDone, taxesAndEqpContent, superscript1Content } = this.props;
    let accessoryReferenceData = [];

    /** To be refactored */
    let bauTileData = {};
    if (this.isMidnight) {
      bauTileData = this.prepareTilesDataClone(accessoryDetails, addOnLoadingDone, this, this.props);
      accessoryReferenceData = getAddonsRefData(accessoryDetails, this.props.plansReferenceData);
    }

    return (
      <>
        <React.Fragment>
          {this.flowType !== 'LTE' && accessoryDetails && accessoryDetails.length > 0 && (
            <>
              <Margin bottom="20px" top="20px">
                <TitleLockup
                  data={{
                    title: {
                      bold: false,
                      size: 'titleSmall',
                      children: title,
                    },
                    subtitle: {
                      size: 'bodyMedium',
                      color: 'primary',
                      children: subTitle,
                    },
                  }}
                />
              </Margin>

              <StreamingServices
                refData={accessoryReferenceData}
                accessory={accessoryDetails}
                removeAddon={this.removeCart}
                addAddon={this.addToCart}
                updateAddonQuantity={(action, qnt, dispname) => this.onRemovingStreamTV(action, qnt, dispname)}
                bauTileData={bauTileData}
                notifyMessage={this.props.notifyMessage}
                buttonSelectState={this.state.buttonSelectState}
                parentProps={this.props}
                taxesAndEqpContent={taxesAndEqpContent}
                superscript1Content={superscript1Content}
              />
            </>
          )}
        </React.Fragment>
      </>
    );
  }
}

StreamingService.propTypes = {
  title: PropTypes.string,
  // invokeTaggingAPI: PropTypes.func,
  accessoryDetails: PropTypes.array,
  cartDetails: PropTypes.object,
  subTitle: PropTypes.string,
  plansReferenceData: PropTypes.object,
  notifyMessage: PropTypes.string,
  getskipStatus: PropTypes.func,
  // showAddNotification: PropTypes.any,
  showAddonNotification: PropTypes.any,
  // selectedProdID: PropTypes.any,
  setSelectedProduct: PropTypes.any,
  postAddToCart: PropTypes.func,
  addOnLoadingDone: PropTypes.bool,
  // showCartApiError: PropTypes.bool,
  // disclaimerText: PropTypes.string,
  addonObj: PropTypes.object,
  homePhoneID: PropTypes.any,
  homeProdId: PropTypes.any,
  handleBtnClick: PropTypes.func,
  npanxx: PropTypes.any,
  // skipBtnCall: PropTypes.any,
  taxesAndEqpContent: PropTypes.string,
  superscript1Content: PropTypes.string,
};
